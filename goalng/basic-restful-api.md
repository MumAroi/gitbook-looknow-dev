# Basic Restful API

บทความนี้จะมาสอนทำ golang basic api กัน

## Project and Structure

เริ่มต้นก็มาสร้าง project กันก่อนเลย

```
mkdir golang-mysql-api
cd golang-mysql-api
```

ต่อไปก็ทำการ set  go module

```
#จากตัวอย่าง
go mod init example.com/greetings 
#แก้ไขเป็นชื่อ project ของเรา
go mod init github.com/MumAroi/golang-mysql-api
```

หลังจากนั้นก็มาทำการ set structure project ของเรากันต่อตามนี้

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
```

ต่อมาก็สร้าง file main.go

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
├── main.go
```

โดยตัว main.go เราจะเขียน code ไว้แค่นี้ก่อน

```
package main

func main() {

}
```

ต่อมาก็มาสร้าง file  core.go โดยจะสร้างไว้ที่ directory api&#x20;

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
│   ├── core.go
├── main.go
```

ภายใน file code.go เราก็จะสร้าง func ที่ชื่อ Run() ไว้แบบนี้

```
package api

func Run() {

}

```

ต่อไปก็กลับมาแก้ไข file main.go นิดหน่อย

> ให้ตัว file main.go ทำการ เรียก func Run() จาก core.go เพี่อทำการ run service server

```
package main

import (
	"github.com/MumAroi/golang-mysql-api/api"
)

func main() {
	// run server
	api.Run()
}
```

หลังจากนั้นก็มาสร้าง file .env เพี่อเอาไว้ config ค่าต่างๆใน project

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
│   ├── core.go
├── .env
├── main.go
```

.env fie

```
# Mysql
DB_HOST=mysql               
DB_DRIVER=mysql 
API_SECRET=2x2H45xf # Used for creating a JWT
DB_USER=mydatebase
DB_PASSWORD=123456
DB_NAME=api
DB_PORT=3306

# App
GO_PORT=8080
```

ถัดมาก็กลับมาเพิ่ม code ในส่วนของ core.go&#x20;

```
package api

import (
	"log"
	"os"
	
	"github.com/gin-gonic/gin"
	"github.com/joho/godotenv"
)

func init() {
	// loads values from .env into the system
	if err := godotenv.Load(); err != nil {
		log.Print("sad .env file found")
	}
}

func Run() {
	// set up route
	router := SetupRouter()

	// start server
	log.Fatal(router.Run(":" + os.Getenv("GO_PORT")))

}

func SetupRouter() *gin.Engine {

	router := gin.Default()
	
	router.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "hello word",
		})
	})

	gin.SetMode(gin.DebugMode)

	return router
}

```

หลังจากเพิ่ม code เสร็จแล้ว ก็ทำการ update package ที่ยังไม่ได้ติดตั้งกันสะหน่อย

```
go mod tidy
go run main.go
```

ที่ก็ลองทำการ test service ของเราด้วยการ

<mark style="color:blue;">`GET`</mark> `http://localhost:8080`

{% tabs %}
{% tab title="200: OK " %}
```javascript
{
    "message": "hello word",
}
```
{% endtab %}
{% endtabs %}

## Database Connection

ถัดมาเราจะมาสร้าง module ที่จะทำหน้าที่เป็นตัว connection กับ mysql database กัน

เริ่มจากการสร้าง file ที่ชื่อ connection.go ไว้ใน /configs/connection.go

```
golang-mysql-api
├── api
│   ├── configs
│   │    ├── connection.go
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
```

ภายใน connection.go ก็จะมี code ประมาณนี้

```
package configs

import (
	"fmt"
	"os"

	"github.com/sirupsen/logrus"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func Connection() *gorm.DB {
	// create channel
	databaseURI := make(chan string, 1)
	
	// config mysql connection
	DBURL := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8&parseTime=True&loc=Local", os.Getenv("DB_USER"), os.Getenv("DB_PASSWORD"), os.Getenv("DB_HOST"), os.Getenv("DB_PORT"), os.Getenv("DB_NAME"))

	databaseURI <- DBURL

	db, err := gorm.Open(mysql.Open(<-databaseURI), &gorm.Config{})

	if err != nil {
		defer logrus.Info("Connection to Database Failed")
		logrus.Fatal(err.Error())
	}

	logrus.Info("Connection to Database Successfully")

	if err != nil {
		logrus.Fatal(err.Error())
	}

	return db
}

```

## &#x20;Model and Seed

ต่อไปก็มาสร้าง model กันสะหน่อย โดยเราจะทำการสร้าง model 2 ตัวด้วยกัน

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   │    ├── user.go
│   │    ├── image.go
│   ├── routes
│   ├── seeds
│   ├── utils
```

code ของ user.go ก็จะประมาณนี้

```
package models

import (
	"errors"
	"html"
	"log"
	"strings"
	"time"

	"github.com/badoux/checkmail"
	"golang.org/x/crypto/bcrypt"
	"gorm.io/gorm"
)

type User struct {
	ID        uint32 `gorm:"primary_key;auto_increment" json:"id"`
	Nickname  string `gorm:"type:varchar(255);not null;unique" json:"nickname"`
	Email     string `gorm:"type:varchar(100);not null;unique" json:"email"`
	Password  string `gorm:"type:varchar(100);not null" json:"password"`
	CreatedAt time.Time
	UpdatedAt time.Time
}

func Hash(password string) ([]byte, error) {
	return bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
}

func VerifyPassword(hashedPassword, password string) error {
	return bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(password))
}

func (u *User) BeforeSave(db *gorm.DB) error {
	hashedPassword, err := Hash(u.Password)
	if err != nil {
		return err
	}
	u.Password = string(hashedPassword)
	return nil
}

func (u *User) Prepare() {
	u.ID = 0
	u.Nickname = html.EscapeString(strings.TrimSpace(u.Nickname))
	u.Email = html.EscapeString(strings.TrimSpace(u.Email))
	u.CreatedAt = time.Now()
	u.UpdatedAt = time.Now()
}

func (u *User) Validate(action string) error {
	switch strings.ToLower(action) {
	case "update":
		if u.Nickname == "" {
			return errors.New("Required Nickname")
		}
		if u.Password == "" {
			return errors.New("Required Password")
		}
		if u.Email == "" {
			return errors.New("Required Email")
		}
		if err := checkmail.ValidateFormat(u.Email); err != nil {
			return errors.New("Invalid Email")
		}

		return nil
	case "login":
		if u.Password == "" {
			return errors.New("Required Password")
		}
		if u.Email == "" {
			return errors.New("Required Email")
		}
		if err := checkmail.ValidateFormat(u.Email); err != nil {
			return errors.New("Invalid Email")
		}
		return nil

	default:
		if u.Nickname == "" {
			return errors.New("Required Nickname")
		}
		if u.Password == "" {
			return errors.New("Required Password")
		}
		if u.Email == "" {
			return errors.New("Required Email")
		}
		if err := checkmail.ValidateFormat(u.Email); err != nil {
			return errors.New("Invalid Email")
		}
		return nil
	}
}

func (u *User) SaveUser(db *gorm.DB) (*User, error) {

	var err error
	err = db.Debug().Create(&u).Error
	if err != nil {
		return &User{}, err
	}
	return u, nil
}

func (u *User) FindAllUsers(db *gorm.DB) (*[]User, error) {
	var err error
	users := []User{}
	err = db.Debug().Model(&User{}).Limit(100).Find(&users).Error
	if err != nil {
		return &[]User{}, err
	}
	return &users, err
}

func (u *User) FindUserByID(db *gorm.DB, uid uint32) (*User, error) {
	var err error
	err = db.Debug().Model(User{}).Where("id = ?", uid).Find(&u).Error
	if err != nil {
		return &User{}, err
	}

	if errors.Is(err, gorm.ErrRecordNotFound) {
		return &User{}, errors.New("User Not Found")
	}

	return u, err
}

func (u *User) UpdateUser(db *gorm.DB, uid uint32) (*User, error) {

	// To hash the password
	err := u.BeforeSave(db)
	if err != nil {
		log.Fatal(err)
	}
	db = db.Debug().Model(&User{}).Where("id = ?", uid).Take(&User{}).UpdateColumns(
		map[string]interface{}{
			"password":  u.Password,
			"nickname":  u.Nickname,
			"email":     u.Email,
			"update_at": time.Now(),
		},
	)
	if db.Error != nil {
		return &User{}, db.Error
	}
	// This is the display the updated user
	err = db.Debug().Model(&User{}).Where("id = ?", uid).Take(&u).Error
	if err != nil {
		return &User{}, err
	}
	return u, nil
}

func (u *User) DeleteUser(db *gorm.DB, uid uint32) (int64, error) {

	db = db.Debug().Model(&User{}).Where("id = ?", uid).Take(&User{}).Delete(&User{})

	if db.Error != nil {
		return 0, db.Error
	}
	return db.RowsAffected, nil
}

```

ส่วนของ image.go ก็ตามข้างล่างนี้เลย

```
package models

import (
	"errors"
	"html"
	"strings"
	"time"

	"gorm.io/gorm"
)

type Image struct {
	ID        uint64 `gorm:"primary_key;auto_increment" form:"id"`
	Name      string `gorm:"type:varchar(255);not null" form:"name"`
	Url       string `gorm:"type:text;not null" form:"url"`
	CreatedAt time.Time
	UpdatedAt time.Time
}

func (i *Image) Prepare() {
	i.ID = 0
	i.Name = html.EscapeString(strings.TrimSpace(i.Name))
	i.Url = html.EscapeString(strings.TrimSpace(i.Url))
	i.CreatedAt = time.Now()
	i.UpdatedAt = time.Now()
}

func (i *Image) Validate(action string) error {
	switch strings.ToLower(action) {
	case "update":
		if i.Name == "" {
			return errors.New("Required Name")
		}
		return nil
	default:
		if i.Name == "" {
			return errors.New("Required Name")
		}
		if i.Url == "" {
			return errors.New("Required Url")
		}
		return nil
	}
}

func (i *Image) SaveImage(db *gorm.DB) (*Image, error) {
	var err error
	err = db.Debug().Model(&Image{}).Create(&i).Error
	if err != nil {
		return &Image{}, err
	}
	return i, nil
}

func (i *Image) FindAllImage(db *gorm.DB) (*[]Image, error) {
	var err error
	images := []Image{}
	err = db.Debug().Model(&Image{}).Limit(100).Find(&images).Error
	if err != nil {
		return &[]Image{}, err
	}

	return &images, nil
}

func (i *Image) FindImageByID(db *gorm.DB, pid uint64) (*Image, error) {
	var err error
	err = db.Debug().Model(&Image{}).Where("id = ?", pid).Take(&i).Error
	if err != nil {
		return &Image{}, err
	}
	return i, nil
}

func (i *Image) UpdateImage(db *gorm.DB, pid uint64) (*Image, error) {

	var err error
	db = db.Debug().Model(&Image{}).Where("id = ?", pid).Take(&Image{}).UpdateColumns(
		map[string]interface{}{
			"name":       i.Name,
			"updated_at": time.Now(),
		},
	)
	err = db.Debug().Model(&Image{}).Where("id = ?", pid).Take(&i).Error
	if err != nil {
		return &Image{}, err
	}
	return i, nil
}

func (i *Image) DeleteImage(db *gorm.DB, pid uint64) (int64, error) {

	var err error

	db = db.Debug().Model(&Image{}).Where("id = ?", pid).Take(&Image{}).Delete(&Image{})
	err = db.Error
	if db.Error != nil {
		if errors.Is(err, gorm.ErrRecordNotFound) {
			return 0, errors.New("Image not found")
		}

		return 0, db.Error
	}
	return db.RowsAffected, nil
}

```

หลังจากนั้น กลับมาแก้ไข file connection.go กันสักหน่อย

ทำการเพิ่ม  code ข้างล่างเข้าไป

```
// migrate table
db.AutoMigrate(&models.User{}, &models.Image{})

// seed data
seeds.Load(db)
```

ตัว connection.go ก็จะเป็นประมาณนี้

```
package configs

import (
	"fmt"
	"os"

	"github.com/MumAroi/golang-mysql-api/api/models"
	"github.com/MumAroi/golang-mysql-api/api/seeds"
	"github.com/sirupsen/logrus"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func Connection() *gorm.DB {
	databaseURI := make(chan string, 1)

	DBURL := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8&parseTime=True&loc=Local", os.Getenv("DB_USER"), os.Getenv("DB_PASSWORD"), os.Getenv("DB_HOST"), os.Getenv("DB_PORT"), os.Getenv("DB_NAME"))

	databaseURI <- DBURL

	db, err := gorm.Open(mysql.Open(<-databaseURI), &gorm.Config{})

	if err != nil {
		defer logrus.Info("Connection to Database Failed")
		logrus.Fatal(err.Error())
	}

	logrus.Info("Connection to Database Successfully")

	// migrate table
	db.AutoMigrate(&models.User{}, &models.Image{})

	// seed data
	seeds.Load(db)

	if err != nil {
		logrus.Fatal(err.Error())
	}

	return db
}

```

ในส่วนของ core.go ก็จะมีการเพิ่ม code ให้สามารถเรียกใช้ database ได้เข้าไปตามนี้

```
package api

import (
	"log"
	"os"
	
	"github.com/MumAroi/golang-mysql-api/api/configs"
	"github.com/gin-gonic/gin"
	"github.com/joho/godotenv"
)

func init() {
	// loads values from .env into the system
	if err := godotenv.Load(); err != nil {
		log.Print("sad .env file found")
	}
}

func Run() {
	// set up route
	router := SetupRouter()

	// start server
	log.Fatal(router.Run(":" + os.Getenv("GO_PORT")))

}

func SetupRouter() *gin.Engine {

	//  load db connection
	db := configs.Connection()

	router := gin.Default()

	gin.SetMode(gin.DebugMode)
	
	router.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "hello word",
		})
	})

	return router
}

```

## Route

ต่อไป มาสร้าง file route เพื่อเอาไว้ใช้ในการจัดการ route path ต่างๆภายใน web siete กัน

เริ่ม จาก สร้าง file ที่ชื่อ api.go  ไว้ใน directory routes แบบนี้

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   │    ├── api.go
│   ├── seeds
│   ├── utils
```

ทำการเพิ่ม code ลงไปตามนี้

```
package routes

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
)

func InitializeRoutes(db *gorm.DB, route *gin.Engine) {

	route.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "hello word",
		})
	})
}

```

หลังจากสร้าง file api.go เสร็จ ก็กลับมาแก้ไข file core.go กันอีกสักหน่อย

โดยจะทำการเพิ่ม code ที่ใช้ในการ load route เข้าไป

```
package api

import (
	"log"
	"os"

	"github.com/MumAroi/golang-mysql-api/api/configs"
	"github.com/MumAroi/golang-mysql-api/api/routes"
	"github.com/gin-gonic/gin"
	"github.com/joho/godotenv"
)

func init() {
	// loads values from .env into the system
	if err := godotenv.Load(); err != nil {
		log.Print("sad .env file found")
	}
}

func Run() {
	// set up route
	router := SetupRouter()

	// start server
	log.Fatal(router.Run(":" + os.Getenv("GO_PORT")))

}

func SetupRouter() *gin.Engine {

	//  load db connection
	db := configs.Connection()

	router := gin.Default()

	gin.SetMode(gin.DebugMode)

	// load routes
	routes.InitializeRoutes(db, router)

	return router
}

```

## Util

คราวนี้มาสร้าง file util เพื่อไว้เป็นตัวช่วยในการจัดการ func ที่ต้องใช้งานซ้ำๆ

โดยจะแบ่งออกเป็น สอง file&#x20;

dotenv.go เอาไว้ใช้เรียกใช้งาน get env value ใน file .env

token.go เอาไว้ใช้ในการจัดการ func jwt token  &#x20;

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
│   │    ├── dotenv.go
│   │    ├── token.go
```

code ข้างใน dotenv.go ก็จะตามนี้

```
package utils

import (
	"os"

	"github.com/joho/godotenv"
)

func GodotEnv(key string) string {
	env := make(chan string, 1)

	godotenv.Load(".env")
	env <- os.Getenv(key)

	return <-env
}
```

อันนี้ code ของ token.go

```
package utils

import (
	"os"
	"time"

	"github.com/golang-jwt/jwt/v4"
)
 
func CreateToken(user_id uint32) (string, error) {
	claims := jwt.MapClaims{}
	claims["authorized"] = true
	claims["user_id"] = user_id
	claims["exp"] = time.Now().Add(time.Hour * 1).Unix() //Token expires after 1 hour
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte(os.Getenv("API_SECRET")))

}
```

## Controller

เสร็จจากการเพิ่ม route ก็มาถึงในส่วนของ controller กันแล้ว

โดยใน project นี้จะทำการสร้าง  controller 3 ตัวได้แก่

* user-controller
* image-controller
* auth-controller

ภายในแต่ละ controller จะประกอบไปด้วย file service.go ดังนี้

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   │    ├── auth-controller
│   │    │    ├── service.go
│   │    ├── image-controller
│   │    │    ├── service.go
│   │    ├── user-controller
│   │    │    ├── service.go
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
```

code ด้านในของแต่ละ file ก็ตามนี้เลย

auth-controller/service.go

```
package authController

import (
	"net/http"

	"github.com/MumAroi/golang-mysql-api/api/models"
	"github.com/MumAroi/golang-mysql-api/api/utils"
	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/bcrypt"
	"gorm.io/gorm"
)

type service struct {
	db *gorm.DB
}

func NewService(db *gorm.DB) *service {
	return &service{db: db}
}

func (s *service) Login(c *gin.Context) {
	var user models.User

	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"error": err.Error(),
		})
		return
	}

	user.Prepare()

	if err := user.Validate("login"); err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"error": err.Error(),
		})
		return
	}

	token, err := s.SignIn(user.Email, user.Password)

	if err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"error": err.Error(),
		})
		return
	}

	c.JSON(http.StatusOK, gin.H{
		"token": token,
	})

}

func (s *service) SignIn(email, password string) (string, error) {

	var err error

	var user models.User

	err = s.db.Debug().Model(models.User{}).Where("email = ?", email).Take(&user).Error
	if err != nil {
		return "", err
	}
	err = models.VerifyPassword(user.Password, password)
	if err != nil && err == bcrypt.ErrMismatchedHashAndPassword {
		return "", err
	}
	return utils.CreateToken(user.ID)
}

```

image-controller/service.go

```
package imageController

import (
	"log"
	"net/http"
	"os"
	"path/filepath"
	"strconv"

	"github.com/MumAroi/golang-mysql-api/api/models"
	"github.com/MumAroi/golang-mysql-api/api/utils"
	"github.com/gin-gonic/gin"
	"github.com/google/uuid"
	"gorm.io/gorm"
)

type service struct {
	db *gorm.DB
}

func NewService(db *gorm.DB) *service {
	return &service{db: db}
}

func (s *service) CreateImage(c *gin.Context) {

	var image models.Image

	db := s.db.Model(&image)

	file, err := c.FormFile("file")

	if err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"message": "The file cannot be received",
			"error":   err.Error(),
		})
		return
	}

	extension := filepath.Ext(file.Filename)

	newFileName := uuid.New().String() + extension

	if _, err := os.Stat("upload"); os.IsNotExist(err) {
		if err := os.Mkdir("upload", os.ModePerm); err != nil {
			log.Fatal(err)
		}
	}

	path, err := filepath.Abs("upload/" + newFileName)

	if err != nil {

		log.Fatal(err)
	}

	if err := c.SaveUploadedFile(file, path); err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{
			"message": "The file is received, so let's save it",
			"error":   err.Error(),
		})
		return
	}

	imageURL := "http://localhost:" + utils.GodotEnv("GO_PORT") + "/images/" + newFileName

	c.Bind(&image)

	image.Url = imageURL

	image.Prepare()

	if err := image.Validate(""); err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"error": err.Error(),
		})
		return
	}

	if result := db.Create(&image); result.Error != nil {
		c.JSON(http.StatusInternalServerError, gin.H{
			"messs": "Can not create image",
			"error": result.Error.Error(),
		})
		return
	}

	c.JSON(http.StatusCreated, image)

}

func (s *service) GetImages(c *gin.Context) {

	var image models.Image

	images, err := image.FindAllImage(s.db)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{
			"messs": "Can not get images",
			"error": err.Error(),
		})
		return
	}

	c.JSON(http.StatusCreated, images)

}

func (s *service) UpdateImage(c *gin.Context) {

	id := c.Param("id")
	uid, err := strconv.ParseUint(id, 10, 64)

	if err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"messs": "Not found id",
			"error": err.Error(),
		})
		return
	}

	var image models.Image

	c.Bind(&image)

	image.Prepare()

	if err := image.Validate("update"); err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"meaage": "Data invalid",
			"error":  err.Error(),
		})
		return
	}

	imageUpdated, err := image.UpdateImage(s.db, uid)

	if err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"meaage": "Update image fail",
			"error":  err.Error(),
		})
		return
	}

	c.JSON(http.StatusCreated, imageUpdated)

}

func (s *service) DeleteImage(c *gin.Context) {

	var image models.Image

	id := c.Param("id")
	uid, err := strconv.ParseUint(id, 10, 64)

	if err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"messs": "Not found id",
			"error": err.Error(),
		})
		return
	}

	_, err = image.DeleteImage(s.db, uid)

	if err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"meaage": "Update image fail",
			"error":  err.Error(),
		})
		return
	}

	c.JSON(http.StatusNoContent, "")
}

```

user-controller/service.go

```
package userController

import (
	"net/http"

	"github.com/MumAroi/golang-mysql-api/api/models"
	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
)

type service struct {
	db *gorm.DB
}

func NewService(db *gorm.DB) *service {
	return &service{db: db}
}

func (s *service) CreateUser(c *gin.Context) {

	var user models.User

	db := s.db.Model(&user)

	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"error": err.Error(),
		})
		return
	}

	user.Prepare()

	if err := user.Validate(""); err != nil {
		c.JSON(http.StatusUnprocessableEntity, gin.H{
			"error": err.Error(),
		})
		return
	}

	if result := db.Create(&user); result.Error != nil {
		c.JSON(http.StatusInternalServerError, gin.H{
			"error": result.Error.Error(),
		})
		return
	}

	c.JSON(http.StatusCreated, user)
}

func (s *service) GetUsers(c *gin.Context) {

	var user models.User

	users, err := user.FindAllUsers(s.db)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{
			"messs": "Can not get users",
			"error": err.Error(),
		})
		return
	}

	c.JSON(http.StatusCreated, users)

}

```

## Middleware

ขั้นตอนนี้จะมาสร้าง middleware&#x20;

โดยเริ่มจากการสร้าง file authjwt.go ไว้ใน directory middleware

ซึ่งเจ้า authjwt.go จะทำหน้าที่เป็นตัวกรอง request ที่เข้ามายัง route

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   │    ├── authjwt.go
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
```

code ของ authjwt.go ก็จะประมาณนี้

```
package middleware

import (
	"fmt"
	"net/http"
	"os"
	"strings"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v4"
)

func AuthorizationMiddleware(c *gin.Context) {
	s := c.Request.Header.Get("Authorization")

	token := strings.TrimPrefix(s, "Bearer ")

	if err := ValidateToken(token); err != nil {
		c.AbortWithStatus(http.StatusUnauthorized)
		return
	}
}

func ValidateToken(token string) error {
	_, err := jwt.Parse(token, func(t *jwt.Token) (interface{}, error) {
		if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
			return nil, fmt.Errorf("unexpected signing method: %v", t.Header["alg"])
		}
		return []byte(os.Getenv("API_SECRET")), nil
	})
	return err
}

```

ถัดมาก็กลับมาแก้ไข file routes/api.go โดยทำการเพิ่ม middleware กับ route ใหม่เข้าไปตามนี้

```
package routes

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"gorm.io/gorm"

	authController "github.com/MumAroi/golang-mysql-api/api/controllers/auth-controller"
	imageController "github.com/MumAroi/golang-mysql-api/api/controllers/image-controller"
	userController "github.com/MumAroi/golang-mysql-api/api/controllers/user-controller"
	middleware "github.com/MumAroi/golang-mysql-api/api/middlewares"
)

func InitializeRoutes(db *gorm.DB, route *gin.Engine) {

	// set route controller
	authC := authController.NewService(db)
	userC := userController.NewService(db)
	imageC := imageController.NewService(db)

	route.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "hello word",
		})
	})

	route.POST("/login", authC.Login)

	// sest middleware
	protected := route.Group("/", middleware.AuthorizationMiddleware)

	protected.GET("/users", userC.GetUsers)

	protected.POST("/users", userC.CreateUser)

	protected.GET("/images", imageC.GetImages)

	protected.POST("/images", imageC.CreateImage)

	protected.DELETE("/images/:id", imageC.DeleteImage)

	protected.PUT("/images/:id", imageC.UpdateImage)

	protected.GET("/images/:name", func(c *gin.Context) {
		name := c.Param("name")
		c.File("upload/" + name)
	})
}

```

## Docker And Docker-compse

หัวข้อนี้จะมาสร้าง envelopment ที่ใช้ในการจำลอง ecosystem ให้กับ project

เริ่มจาก Dockerfile

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
├── Dockerfile
```

core ข้างในก็ประมาณนี้

```
# base image
FROM golang:alpine as builder

# maintainer 
LABEL maintainer="pope.pepo@gmail.com"

# install git
RUN apk update && apk add --no-cache git

# working directory 
WORKDIR /app

# copy go mod and sum
COPY go.mod go.sum ./

# download go dependencies
RUN go mod download 

#  copy source code
COPY . .

# build go
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# start a new stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates

# working directory 
WORKDIR /root/

# Copy the Pre-built binary file from the previous stage
COPY --from=builder /app/main .
COPY --from=builder /app/.env .     

# Expose port 8080 to the outside world
EXPOSE 8080

# executable
CMD ["./main"]

```

เสร็จแล้วก็มาสร้าง file docker-compose.yml เพื่อจัดการให้ docker อีกที

```
golang-mysql-api
├── api
│   ├── configs
│   ├── controllers
│   ├── middlewares
│   ├── models
│   ├── routes
│   ├── seeds
│   ├── utils
├── Dockerfile
├── docker-compose.yml
```

code ของ docker-compose.yml ก็ประมาณนี้

```
version: '3'
services:
    app:
        container_name: goalng_app
        build: .
        ports:
            - 8080:8080
        restart: on-failure
        volumes:
            - api:/usr/src/app/
        depends_on:
            - mysql
        networks:
            - appnet

    mysql:
      image: mariadb:10.5.8
      container_name: mysql
      ports: 
        - 3306:3306
      environment: 
        - MYSQL_ROOT_HOST=${DB_HOST} 
        - MYSQL_USER=${DB_USER}
        - MYSQL_PASSWORD=${DB_PASSWORD}
        - MYSQL_DATABASE=${DB_NAME}
        - MYSQL_ROOT_PASSWORD=${DB_PASSWORD}
      volumes:
        - database_mysql:/var/lib/mysql
      networks:
        - appnet

    phpmyadmin:
      image: phpmyadmin/phpmyadmin
      container_name: phpmyadmin
      depends_on:
        - mysql
      environment:
        - PMA_HOST=mysql
        - PMA_USER=${DB_USER}
        - PMA_PORT=${DB_PORT}
        - PMA_PASSWORD=${DB_PASSWORD}
      ports:
        - 9090:80
      restart: always
      networks:
        - appnet

volumes:
    api:
    database_mysql:

# Networks for communication between containers
networks:
    appnet:
        driver: bridge
```

เสร็จแล้วก็ลอง run&#x20;

```
docker-compose up -d
หรือ 
docker-compose up --build -d
```

หลังจาก build docker เสร็จก็ลอง เข้า web api ได้ตาม url นี้เลย

```
http://localhost:8080
```

ถ้าอยากลองเล่น api ก็เข้าไปลองเล่นได้ตาม postman นี้เลย

```
https://www.getpostman.com/collections/f8506b1da6e5dfd08dd2
```

> อันนี้เป็น link project แบบเต็มๆ : [https://github.com/MumAroi/golang-mysql-api](https://github.com/MumAroi/golang-mysql-api)

The end.
