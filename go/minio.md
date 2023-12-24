# minio


## docker 部署方式

```bash

docker run -dt                                  \
  -p 9000:9000 -p 9090:9090                     \
  -v PATH:/mnt/data                             \
  -v /etc/default/minio:/etc/config.env         \
  -e "MINIO_CONFIG_ENV_FILE=/etc/config.env"    \
  --name "minio_local"                          \
  minio server --console-address ":9090"


```

* 我选择了docker-compose的管理方式

```yml
version: '3'

services:
  server:
    image: minio/minio
    command: server --console-address ":9001" /data
    environment:
      MINIO_ROOT_USER: user
      MINIO_ROOT_PASSWORD: password
      MINIO_BROWSER_REDIRECT_URL: http://localhost:9001
      MINIO_SERVER_URL: http://localhost:9000
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
    volumes:
      - D:\desktop\container\minio-docker\.data:/data
    ports:
      - "9000:9000"
      - "9001:9001"

volumes:
  data:

```
<p></p>


> 打开服务的 9001 端口服务输入账号密码登录控制台


![minio创建buckets](https://cdn.jsdelivr.net/gh/sigam-Zq/picStore/docs20231223091515.png)


## go sdk 引入


* 在项目中
```bash
go get github.com/minio/minio-go/v7

```

[minio 官方文档](https://min.io/docs/minio/linux/developers/go/minio-go.html)


### 导入后封装在util的工具类

```目录

└─utility
        .gitkeep
        abb
        jwtutil.go
        jwtutil_test.go
        minioclient.go
        minioclient_test.go


```

> miniclient.go
```go
package utility

import (
	"context"

	"github.com/gogf/gf/v2/frame/g"
	"github.com/minio/minio-go/v7"
	"github.com/minio/minio-go/v7/pkg/credentials"
)

var (
	mClient *minio.Client
)

func GetMinioClient(ctx context.Context) (*minio.Client, error) {

	if mClient == nil {

		end, err := g.Cfg().Get(ctx, "minio.endpoint")
		if err != nil {
			g.Log().Errorf(ctx, "GetMinioClient err %s \n", err)
			return nil, err
		}
		acc, err := g.Cfg().Get(ctx, "minio.accessKeyID")
		if err != nil {
			g.Log().Errorf(ctx, "GetMinioClient err %s \n", err)
			return nil, err
		}

		ser, err := g.Cfg().Get(ctx, "minio.secretAccessKey")
		if err != nil {
			g.Log().Errorf(ctx, "GetMinioClient err %s \n", err)
			return nil, err
		}

		useSSL, err := g.Cfg().Get(ctx, "minio.useSSL")
		if err != nil {
			g.Log().Errorf(ctx, "GetMinioClient err %s \n", err)
			return nil, err
		}

		mClient, err = minio.New(end.String(), &minio.Options{
			Creds:  credentials.NewStaticV4(acc.String(), ser.String(), ""),
			Secure: useSSL.Bool(),
		})
		if err != nil {
			g.Log().Errorf(ctx, "GetMinioClient err %s \n", err)
			return nil, err
		}

	}
	return mClient, nil
}

func GetDefaultBucket(ctx context.Context) string {

	bucketName, err := g.Cfg().Get(ctx, "minio.defaultBucket")
	if err != nil {
		g.Log().Errorf(ctx, "GetDefaultBucket err %s \n", err)
		g.Log().Errorf(ctx, " 未配置默认bucket 桶 \n")
		return ""
	}
	return bucketName.String()

}

```


### 测试用例


> miniclient_test.go

```go
package utility

import (
	"context"
	"testing"

	"github.com/minio/minio-go/v7"
)

func TestSetFile(t *testing.T) {
	ctx := context.Background()

	bucketName := GetDefaultBucket(ctx)

	c, err := GetMinioClient(ctx)
	if err != nil {
		panic(err)
	}
	contentType := "application/octet-stream"

	info, err := c.FPutObject(ctx, bucketName, "testfile2", "./abb", minio.PutObjectOptions{ContentType: contentType})
	if err != nil {
		panic(err)
	}

	t.Logf("upinfo %v \n", info)
	//  upinfo {crmobject testfile d41d8cd98f00b204e9800998ecf8427e 0 0001-01-01 00:00:00 +0000 UTC   0001-01-01 00:00:00 +0000 UTC     }

}

func TestListFile(t *testing.T) {
	ctx := context.Background()

	bucketName := GetDefaultBucket(ctx)

	c, err := GetMinioClient(ctx)
	if err != nil {
		panic(err)
	}
	objChan := c.ListObjects(ctx, bucketName, minio.ListObjectsOptions{})
	if err != nil {
		panic(err)
	}

	t.Logf("ListObjects  objChan %v \n", <-objChan)
	// ListObjects  objChan {d41d8cd98f00b204e9800998ecf8427e testfile 2023-12-23 02:50:03.271 +0000 UTC 0  0001-01-01 00:00:00 +0000 UTC map[] map[] map[] 0
	// 	{{http://s3.amazonaws.com/doc/2006-03-01/ Owner} 02d6176db174dc93cb1b899f7c6078f08654445fe8cf1b6ce98d8855f66bdbf4 minio} [] STANDARD false false   false 0001-01-01 00:00:00 +0000
	// 	UTC  <nil>     <nil> <nil>}

	binfo, err := c.ListBuckets(ctx)

	for _, bucket := range binfo {
		t.Logf("ListBuckets b info :   %v", bucket)

		// minioclient_test.go:50: ListBuckets b info :   {box-im 2023-11-02 06:57:54.339 +0000 UTC}
		// minioclient_test.go:50: ListBuckets b info :   {crmobject 2023-12-23 01:18:29.317 +0000 UTC}
	}

}

func TestGetFile(t *testing.T) {
	ctx := context.Background()

	bucketName := GetDefaultBucket(ctx)

	c, err := GetMinioClient(ctx)
	if err != nil {
		panic(err)
	}
	obj, err := c.GetObject(ctx, bucketName, "testfile", minio.GetObjectOptions{})
	if err != nil {
		panic(err)
	}

	t.Logf("getObject: %v \n", obj)
	// getObject: &{0xc00027c008 0xc00027e000 0xc00027e060 0xc00024e050 0x3a5360 0 {  {0 0 <nil>} 0  {0 0 <nil>} map[] map[] map[] 0 {{ }  } []  false false
	//  false {0 0 <nil>}  <nil>     <nil> <nil>} false false false <nil> false false}

	// download? 下载
	err = c.FGetObject(ctx, bucketName, "testfile", "../myObject", minio.GetObjectOptions{})
	if err != nil {
		panic(err)
	}

}

```