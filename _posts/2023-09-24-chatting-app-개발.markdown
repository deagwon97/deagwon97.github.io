---
layout: post
thumbnail: 5dafabf7-2818-4604-861c-d3c754fb825f
title: "chatting app 개발"
createdAt: 2023-09-24 13:09:59.198000
updatedAt: 2023-09-24 13:09:59.198000
category: "백앤드"
---
토이 프로젝트로 익명 채팅방을 만들었습니다. websocket을 통해서 사용자가 서버에 메시지을 보내면 서버는 다른 사용자에게 메시지를 전달하는 간단한 프로젝트 입니다. 채팅 앱은 bdg.blog 소스코드에 포함되어 있습니다. 

- 앱 링크: [bdg.chat](https://deagwon.com/chat)
- github: https://github.com/deagwon97/bdg-blog-v2/

<img alt="image" src="/images/f370aa56-4261-4cbc-b0ac-c390168b9cd5"/>

### 기능 명세

우선 구체적인 기능을 글로 정리했습니다.

- 사용자는 채팅방에 입장하는 순간 랜덤하게 이름을 부여받는다.
- 사용자가 채팅방에 입장하면 기존에 채팅방에 입장한 사람들은 “XX가 입장했습니다.”라는 메세지를 받는다.
- 사용자가 텍스트를 입력하고 엔터키 혹은 전송 버튼을 누르면 메시지가 서버로 전달된다.
- 서버는 사용자가 전송한 메시지와 사용자의 이름을 다른 사용자들에게 전송한다.
- 사용자가 브라우저는 닫거나 다른 페이지로 이동하면 기존에 채팅방에 입장한 사람들은 “XX가 나갔습니다.”라는 메시지를 받는다.

### 고려사항

bdg blog는 2개의 replicaSet을 가지는 deployment으로 배포됩니다. 서로 다른 pod가 채팅방의 메시지를 공유하기 위해서는 메시지 브로커가 필요합니다.

<img alt="image" src="/images/5dafabf7-2818-4604-861c-d3c754fb825f"/>

## 구현

### 서버

서버가 실행되는 시점에 메시지 브로커 redis와 연결되는 publisher객체와 subscriber객체를 싱글톤으로 생성합니다. 이후 서버는 클라이언트의 web socket connection 생성하는 요청을 기다립니다.

``````tsx
// server/chat/redisSingleton.ts
import { Redis } from 'ioredis'

const globalRedis = globalThis as unknown as {
  redisPub: Redis | undefined
  redisSub: Redis | undefined
}

export const redisPub =
  globalRedis.redisPub ??
  new Redis({
    host: process.env.NEXT_PUBLIC_REDIS_HOST
  })

export const getRedisSub = async () => {
  if (globalRedis.redisSub) {
    return globalRedis.redisSub
  } else {
    globalRedis.redisSub = new Redis({
      host: process.env.NEXT_PUBLIC_REDIS_HOST
    })
    await globalRedis.redisSub.subscribe('bdg-chat-user-data', (err, count) => {
      if (err) {
        console.log(``error: Subscribed to NULL channels. NULL``)
      }
    })
    return globalRedis.redisSub
  }
}
``````

``````tsx
// server/chat/chatSocketHandler.ts
import { NextApiRequest } from 'next'
import { Server } from 'socket.io'
import { NextApiResponseWithSocket } from 'socket.d'
import { redisPub, getRedisSub } from 'server/chat/redisSingleton'
import Redis from 'ioredis'

let io: Server | null = null
let redisSub: Redis | null = null

const ChatSocketHandler = async (
  _: NextApiRequest,
  res: NextApiResponseWithSocket
) => {
  if (!res.socket.server.io) {
    if (!io) {
      io = new Server(res.socket.server)
    }
    res.socket.server.io = io
    let userName = ''

    // create websocket connection
    // when new client connects
    io.on('connection', async (socket) => {
      // create redis sub
      redisSub = await getRedisSub()

      socket.on('client-server-chat', (msg) => {
        userName = msg?.userName as string
        redisPub.publish(
          'bdg-chat-user-data',
          JSON.stringify({
            ...{
              message: msg
            }
          })
        )
      })

      socket.on('disconnect', () => {
        redisPub.publish(
          'bdg-chat-user-data',
          JSON.stringify({
            ...{
              message: {
                type: 'notice',
                userName: userName,
                message: ``NULL님이 나갔습니다.``
              }
            }
          })
        )
        socket.disconnect()
        return
      })

      redisSub.on('message', (_, message) => {
        const redisMessage = JSON.parse(message)
        socket.emit('server-client-chat', redisMessage.message)
      })
    })
  }
  res.end()
}

export default ChatSocketHandler
``````

### 클라이언트

사용자가 채팅방에 입장하는 순간 이름을 서버로부터 이름을 부여받습니다. 이후 webSocket을 연결하고, 부여받은 이름을 통해서 입장 메시지를 서버에 전달합니다. 

다음부터는 ``socket.emit`` 함수와 ``socket.on`` 함수를 사용해 서버로부터 메시지를 주고받도록 구현했습니다.

``````tsx
const initSocketCallback = useCallback(async () => {
  if (userName === '') {
    return
  }
  await fetch('/api/socket')
  socket = io('', {
    path: '/socket.io',
    transports: ['websocket'],
    secure: process.env.NODE_ENV === 'production'
  }) as Socket

  socket.emit('client-server-chat', {
    type: 'notice',
    userName: userName,
    message: ``NULL님이 입장했습니다.``
  } as ChatMessage)

  socket.on('server-client-chat', (msg) => {
    setChatMessageList((chatMessageList) => [...chatMessageList, msg])
  })
	}, [userName]
)

useEffect(() => {
  initSocketCallback()
}, [initSocketCallback])

const sendChat = () => {
  if (input === '') {
    return
  }
  if (socket === null) {
    initSocketCallback()
    return
  }
  socket.emit('client-server-chat', {
    type: 'chat',
    userName: userName,
    message: input
  } as ChatMessage)
  setInput('')
}
``````

### Reference

- https://github.com/redis/ioredis
- https://github.com/socketio/socket.io-client

