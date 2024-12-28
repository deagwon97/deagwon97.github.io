---
layout: post
thumbnail: 55385390-d718-457d-a0ac-674df254f61e
title: "[Go] Gin Web Framework CRUD 맛보기."
createdAt: 2023-07-26 10:43:37.367000
updatedAt: 2023-07-26 10:43:37.367000
category: "백앤드"
---
## Intro

<img alt="image" src="/images/55385390-d718-457d-a0ac-674df254f61e"/>

> **Gin Web Framework**
Gin is a web framework written in Go (Golang). It features a martini-like API with performance that is up to 40 times faster thanks to httprouter. If you need performance and good productivity, you will love Gin.

 Hands-On Full-Stack Development with Go 책을 참고하여 Go의 CRUD web api를 만들어보았다.
 Pagination 및 Swagger를 추가한 코드는 git에서 확인가능하다.
 
>  https://github.com/deagwon97/go-gin-crud
 

``````go
src
├── app
│   ├── models
│   │   └── models.go
│   ├── dblayer
│   │   ├── dblayer.go
│   │   └── orm.go
│   └── rest
│       ├── handler.go
│       └── rest.go
│
├── app2
├── app3
├── ...
│
├── database
│   └── database.go
├── routes
│   └── routes.go
├── main.go
│
├── go.mod
└── go.sum
``````

## app
 위에서 언급한 책에서는 여러 로직이 섞여서 프로젝트가 구성되어있었다. Django의 구조를 참고하여 로직별로 app을 분리하였다. app 폴더에는 models, dblayer rest 이렇게 3개의 package가 존재한다.
 
### 1. models : 모델 정의 django의 models.py와 대응된다.
  - models.go
  ``````go
    package models

    // Content 구조체 정의
    type Content struct {
        // gorm.Model // gorm.Model은 db구조를 변형한다.
        ID        int    ``gorm:"column:content_id" json:"content_id"``
        Title     string ``gorm:"column:title"      json:"title"``
        Summary   string ``gorm:"column:summary"    json:"summary"``
        Content   string ``gorm:"column:content"    json:"content"``
        CreatedAt string ``gorm:"column:created_at" json:"created_at"``
        UpdatedAt string ``gorm:"column:updated_at" json:"updated_at"``
        User      int    ``gorm:"column:user"       json:"user"``
    }

    // Content 함수 정의
    // gorm에서 호출하는 테이블 명  커스텀
    // 기본값 Content -> contents
    func (Content) TableName() string {
        return "content_content"
    }
    ``````
        
            
- dblayer : 데이터 베이스와 관련된 코드
  - dblayer.go
    - 데이터베이스 레이어의 모든 동작을 정의하는 인터페이스이다.
    - 데이터베이스 레이어 밖의 모든 데이터베이스 관련 코드는 이 인터페이스의 메서드만을 사용한다.
            
    ``````go
	package dblayer
            
	import "go-api/content/models"
            
	// DBLayer 인터페이스 정의
	type DBLayer interface {
		GetAllContents() ([]models.Content, error)
	}
	``````        
  - orm.go
  ``````go	
    package dblayer
            
    import (
            "database/sql"
            
            "go-api/content/models"
            
            "gorm.io/driver/mysql"
            "gorm.io/gorm"
    )
    // ----------------------------------
    // DBORM 구조체 정의
    // *gorm.DB 타입을 임베드
    type DBORM struct {
        *gorm.DB
    }

    // DBORM 생성자 정의
    func NewORM(dbengine string, dsn string) (*DBORM, error) {
        sqlDB, err := sql.Open(dbengine, dsn)
        // gorm.Open은 *gorm.DB 타입을 초기화한다.
        gormDB, err := gorm.Open(mysql.New(mysql.Config{
            Conn: sqlDB,
        }), &gorm.Config{})

        return &DBORM{
            DB: gormDB,
        }, err
    }

    // DBORM 함수 정의
    func (db *DBORM) GetAllContents() (contents []models.Content, err error) {
        return contents, db.Find(&contents).Error
    }

    // ----------------------------------
    ``````

            
### 2. rest
- handler.go 
  - 클라이언트의 요청을 처리
  - django의 views.py에 대응한다.
  - 데이터베이스의 crud와 관련된 함수는 dblayer interface에서 정의된 함수를 사용한다.
  - 인증 및 권한, 오류 코드 처리 등과 같은 로직이 추가된다.
           
           
 ``````go
package rest
import (
    "go-api/content/dblayer"
    "go-api/database"

    "strconv"
    "net/http"

    "github.com/gin-gonic/gin"
)
// Handler Class 정의 -----------------
// Handler 구조체 정의
type Handler struct {
    db dblayer.DBLayer
}
// Handler 구조체의 생성자는 정의하지 않음
// Handler 함수 정의
func (h *Handler) GetContents(c *gin.Context) {
    if h.db == nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "dsn 오류"})
        return
    }
    contents, err := h.db.GetAllContents() // dblayer의 함수 호출
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusOK, contents)
}
// ----------------------------------
// HandlerInterface 정의
type HandlerInterface interface {
    // HandlerInterface는 다음과 같은 함수를 갖는다.
    GetContents(c *gin.Context)
    GetContent(c *gin.Context)
}
// HandlerInterface 생성자 정의
func NewHandler() (HandlerInterface, error) {
    dsn := database.DataSource
    // DBORM 초기화 - DBORM 생성자 호출
    db, err := dblayer.NewORM("mysql", dsn)
    if err != nil {
        return nil, err
    }
    return &Handler{
        db: db,
    }, nil
}
// ----------------------------------
 ``````


- rest.go
  - RESTfull AP의 entrypoint function 선언부이다.
    - handler.go에서 정의한 handler 와 entrypoint를 연결한다.
    - *django의 urls.py와 대응한다.*
    
    ``````go
    package rest
    
    import "github.com/gin-gonic/gin"
    
    func AddContentRoutes(rg *gin.RouterGroup) {
        content := rg.Group("/content")
        h, _ := NewHandler()
        content.GET("/list", h.GetContents)
        content.GET("/:id", h.GetContent)
    }
    ``````

## main

### 1. database
  - database.go
  	데이터 베이스 접근과 관련된 코드를 포함한다. 환경변수의 값을 참조하여 프로젝트 내부에서 db에 접근하는 것을 편리하게 도와준다.
  
### 2. routes
  - routes.go
  	여러 app속에 들어있는 rest를 묶어서 하나의 route를 생성한다. 
	baseUrl과 swagger 추가할 수 있다.
    ``````go
    package routes

    import (
        "github.com/gin-gonic/gin"
        content_rest "go-api/content/rest"
    )

    func Run(address string) error {
        router := gin.Default()
        v1 := router.Group("/")
        content_rest.AddContentRoutes(v1)
        return router.Run(address)
    }
    ``````
        
### 3. main
- main.go
    ``````go
    func main() {
    	routes.Run(":8000")
    }
    ``````
    
## Reference
- Hands-On Full-Stack Development with Go
- https://github.com/gin-gonic/gin
