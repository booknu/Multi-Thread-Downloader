[TOC]

# Usage

# Project Goals

## Functions

### Download Functions

1. **Simultaneous download** for multiple download tasks
2. Download streaming data consisting of **chunk files**
3. Download single file by splitting it into **multiple parts**

### Managing Functions

1. Control download tasks by **pause, release, cancel**

### Monitoring Functions

1. Monitoring **total file size, recent download speed, average download speed, highest download speed, errors**
2. Download status for each chunks, multi-parts

### Logging and Restoring Errors Functions

1. Logging download status
2. Make download restart possible with log file
   - If use restoring option, download task retry to re-download for all failed part file download

### Scalability

#### Listener

```java
public interface DownloadListener {
    public void extracted();
    public void downloadStarted();
    public void errorDetected(DownloadError e);
    public void downloadFinished();
    public void concatFinished();
    public void postProcess();
    public void postProcessFinished();
}
```

# Project Structure

## Task Hierarchy

![](/readme-imgs/Task Hierarchy.png)

## Download Flow



## Class Relations

### Visualization

### DownloadManager

- purpose
  - Accept request for add download task

### DownloadPool

- 각 파일 계층이 아닌 여기서만 스레딩하면 생기는 문제점: Chunk를 Single Connection으로 다운로드 받고 싶을 때 다운로드 받지 못하고, 다른 DownloadTask에 우선순위가 밀려 기아 현상 발생 가능
  - 기아 현상이 길어질수록 다운로드 성공 가능성이 낮아짐
- 만약 계층별로 스레딩 할 경우 모든 스레드 풀의 fixed 값을 곱해서 64 이상 나오면 Warning 출력하기
- purpose
  - Multi-connection threading to download part file data

### TaskLog

- purpose
  - Status log for download task
  - Restarting download is possible when using log data's information

- data

  - ```
    {
        "chunkUrls": [url array],
        "saveFilePath": "file path",
        "status": "status" // working, done error
        "downloadStatus": [ // array for each chunk tasks
            {
                "beginOffset": long, // chunk data's [begin, end) offset in file
                "endOffset: long,
                "parts": [ // array for each part data tasks
                    {
                        "beg": long, // [beg, end) offset in file
                        "end": long,
                        "done": long,
                        "status": "status" // working, done, error
                    }, // another part file data...
                ]
            }, // another chunk data...
        ]
    }
    ```

- functions

  - Save log to file data
  - Read log from file data
  - Make download restart possible with log file

### DownloadTask

- purpose

  - Collect informations for download task

- data

  - ```
    {
    	"chunkUrls": [url array],
    	"saveFilePath": "file path",
    	"speedInfo": SpeedInfo,
    	"log": TaskLog
    }
    ```

- functions

  - Create download task information
  - Read restart data from log

DownloadManager - 스레드 풀 관리
addDownload(urls, path, postProcessor)

DownloadLog - 로깅
파일 - 분할파일 - 파트파일 - 버퍼단위
파일 내 offset 정보 저장
offset [start, end)

DownloadTask - 정보 저장용, 속도 정보 등등?
extract?

TaskInfo - 속도, 크기, 상태

DownloadWorker - 분할파일에 대해 관리
download(task, postProcessor)

TinyWorker - 단일 url에 대한 다운로드

PartWorker - 단일 파트에 대한 다운로드

PostProcess - interface
process(file, newFile)


log마다 download settings 정보도 들어가야 함

File
Divided
Part
Buff
구조