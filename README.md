# FFmpeg-Web-Client  

[![made-with-Go](https://img.shields.io/badge/Made%20with-Go-1f425f.svg)](https://golang.org/)

윈도우 서비스에 등록된 프로그램이 서버 역할을 하여 웹 클라이언트의 자원을 이용해 영상을 트랜스코딩(인코딩)하고, 동시에 ftp 서버로 전송하는 기능을 한다. 

## Browsers support

| [<img src="https://raw.githubusercontent.com/alrra/browser-logos/master/src/edge/edge_48x48.png" alt="IE / Edge" width="24px" height="24px" />](http://godban.github.io/browsers-support-badges/)</br>IE / Edge | [<img src="https://raw.githubusercontent.com/alrra/browser-logos/master/src/chrome/chrome_48x48.png" alt="Chrome" width="24px" height="24px" />](http://godban.github.io/browsers-support-badges/)</br>Chrome |
| --------- | --------- |
| IE10, IE11| last version

## Install

    go get github.com/tj3828/ffmpeg-web-client

## Run

1. 해당 프로그램을 실행시키기 위해서는 해당 라이브러리를 다운받아야 한다.


    #### for service

    * [golang/sys](https://godoc.org/golang.org/x/sys) : windows service
    * [gorilla/websocket](https://github.com/gorilla/websocket) : websocket 
    * [jinzhu/copier](https://github.com/jinzhu/copier) : deep copy
    * [dutchcoders/goftp](https://github.com/dutchcoders/goftp) : ftp client
    * [ffmpeg.exe / ffprobe.exe](https://ffmpeg.zeranoe.com/builds/) : ffmpeg and ffprobe .exe file 

    #### for test

    * [goftp/server](https://github.com/goftp/server) : ftp server for test
    

2. ffmpeg.exe / ffprobe.exe를 server.go 디렉토리로 위치 변경

3. 테스트용 ftp server 실행

        exampleftpd -root /tmp

4. ftp server 확인

        ftp > open localhost 2121

5. sample.html의 웹소켓 연결 확인 

## API Reference

* ftp server 

    | Domain      | Port           | 
    | ----------  | ---------------|
    | /           | 2121           | 

* default : localhost:5050

* websocket

    * Request 

            Get /echo

    * Progress Json Data

        | Field            | Description                            | Optional   |
        | ---------------- | ---------------------------------------| ---------- |
        | `fileIndex`      | 파일 인덱스                             | no         |
        | `fileName`       | 파일 이름                               | yes        |
        | `progressTime`   | 현재 인코딩 진행된 시간                  | yes        |
        | `progressStatus` | 인코딩 진행 상황 (continue / end)        | no        |
        | `totalTime`      | 파일의 총 시간 길이                      | yes        |
        | `progressRate`   | 인코딩 진행률 ( 00.00)                  | no         |

* ftp 서버의 파일 존재 유무

    * Request

            Get /fileExist

        Query parameters :

        | Field            | Description                            | Optional   |
        | ---------------- | ---------------------------------------| ---------- |
        | `uploadFileName` | 확인할 파일의 명(인코딩된 파일명)         | no         |
        | `courseCd`       | 과목코드                                | no         |

    * Response

            Content-Type : text/plain;
            exmaple : true(없음) / false(있음) / error

* 트랜스코딩 요청

    * Request

            Post /transcoding
            Content-Type : multipart/form-data; 
    

       - Form Data :

            | Field          | Description                            | Optional   |
            | -------------- | ---------------------------------------| ---------- |
            | `uploadFileData` | 실제 파일 데이터                      | no        |
            | `uploadFileInfo` | 'uploadFileData'의 정보               | no        |
            | `uploadEncodingInfo`| 인코딩 정보                        | no        |

       - 'uploadFileInfo' Json Data :

            | Field             | Description                               | Optional   |
            | ----------------- | ------------------------------------------| ---------- |
            | `fileIndex`       | 요청 파일의 고유 인덱스                     | no         |
            | `courseCd`        | 과목코드                                   | no         |
            | `courseNo`        | 순번(파일명)                               | no         |
            | `chapterNo`       | 장/절번호                                  | yes        |
            | `courseName`      | 장/절명                                    | yes        |
            | `courseDetailName`| 차시명                                     | yes        |
            | `uploadFileName`  | 파일명 (순번 + 인코딩 확장자)                | no         |
            | `uploadFileSize`  | 파일 사이즈                                 | yes        |
            | `uploadFileExt`   | 기존 파일 확장자                            | yes        |

       - 'uploadEncodingInfo' Json Data :

            | Field             | Description                               | Optional   |
            | ----------------- | ------------------------------------------| ---------- |
            | `videoCodec`      | 비디오 코덱                                | no         |
            | `videoBitrate`    | 비디오 비트레이트                           | no         |
            | `audioCodec`      | 오디오 코덱                                | no         |
            | `audioBitrate`    | 오디오 비트레이트                           | no        |
            | `resolution`      | 해상도                                     | no        |

    * Response

            Content-Type : text/plain;

* 트랜스코딩 중지

    * Request

            DELETE /transcoding
    
       - Form Data :

            | Field          | Description                            | Optional   |
            | -------------- | ---------------------------------------| ---------- |
            | `fileIndex`    | 삭제할 파일 인덱스                       | no        |

    * Response

             Content-Type : text/plain;
             exmaple : true / false

* 서비스 프로그램 실행 여부 체크

    * Request

            Get /serviceCheck

    * Response

            Content-Type : text/plain;
            exmaple : 1(running) / error


## Notice

* 서비스 실행되는 과정을 로그로 찍고 싶다면, Run() 안에서 Init() 호출
* 웹의 파일 경로 보안이슈(fakepath)로 인해 기본 동작은 웹 클라이언트로부터 받은 데이터를 로컬에 저장한 후 이를 인코딩하여 ftp 서버로 전송한다. 그렇기 때문에 이 과정에서 로컬에 저장한 데이터와 인코딩 된 데이터를 지우는 과정이 필요하다.
* 중지 기능은 인코딩 중에는 가능하지만, 업로딩 중에는 사용할 수 없다.


* ftp 서버의 파일 구조

        /GoTest/과목코드/순번.mp4
        
## References

 * [ffmpeg command](https://ffmpeg.org/ffmpeg.html)
