<div align="center">

# dashscope-go-sdk

**Go SDK for Alibaba Cloud DashScope API**

*Supports Qwen LLM, Qwen-VL, Qwen-Audio, Wanx image generation, and Paraformer speech recognition*

<br/>

[![Build](https://github.com/the-open-agent/dashscope-go-sdk/workflows/Build/badge.svg?style=flat-square)](https://github.com/the-open-agent/dashscope-go-sdk/actions/workflows/build.yml)
[![Release](https://img.shields.io/github/v/release/the-open-agent/dashscope-go-sdk?style=flat-square&color=4f46e5)](https://github.com/the-open-agent/dashscope-go-sdk/releases/latest)
[![Go Reference](https://pkg.go.dev/badge/github.com/the-open-agent/dashscope-go-sdk.svg)](https://pkg.go.dev/github.com/the-open-agent/dashscope-go-sdk)
[![Go Report](https://goreportcard.com/badge/github.com/the-open-agent/dashscope-go-sdk?style=flat-square)](https://goreportcard.com/report/github.com/the-open-agent/dashscope-go-sdk)
[![License](https://img.shields.io/github/license/the-open-agent/dashscope-go-sdk?style=flat-square&color=22c55e)](https://github.com/the-open-agent/dashscope-go-sdk/blob/master/LICENSE)

</div>

---

## What is this?

`dashscope-go-sdk` is a Go client for [Alibaba Cloud DashScope](https://dashscope.aliyun.com/), providing idiomatic Go APIs for:

- **Qwen** — text generation (streaming & non-streaming)
- **Qwen-VL** — vision-language understanding
- **Qwen-Audio** — audio-language understanding
- **Wanx** — text-to-image generation
- **Paraformer** — real-time speech recognition (ASR)

## Install

```bash
go get github.com/the-open-agent/dashscope-go-sdk
```

## Quick Start

Set your API key first ([create one here](https://help.aliyun.com/zh/dashscope/developer-reference/activate-dashscope-and-create-an-api-key)):

```bash
export DASHSCOPE_API_KEY=your-api-key
```

## Usage

### Text Generation (Qwen)

```go
import (
    "context"
    "fmt"
    "os"

    dashscopego "github.com/the-open-agent/dashscope-go-sdk"
    "github.com/the-open-agent/dashscope-go-sdk/qwen"
)

func main() {
    token := os.Getenv("DASHSCOPE_API_KEY")
    cli := dashscopego.NewTongyiClient(qwen.QwenTurbo, token)

    content := qwen.TextContent{Text: "Tell me a joke"}
    input := dashscopego.TextInput{
        Messages: []dashscopego.TextMessage{
            {Role: "user", Content: &content},
        },
    }

    // Optional: streaming callback
    streamCallbackFn := func(ctx context.Context, chunk []byte) error {
        fmt.Print(string(chunk))
        return nil
    }

    req := &dashscopego.TextRequest{
        Input:       input,
        StreamingFn: streamCallbackFn,
    }

    resp, err := cli.CreateCompletion(context.TODO(), req)
    if err != nil {
        panic(err)
    }

    fmt.Println("\nFull response:", resp.Output.Choices[0].Message.Content.ToString())
}
```

### Vision-Language (Qwen-VL)

```go
import (
    "context"
    "fmt"
    "os"

    dashscopego "github.com/the-open-agent/dashscope-go-sdk"
    "github.com/the-open-agent/dashscope-go-sdk/qwen"
)

func main() {
    cli := dashscopego.NewTongyiClient(qwen.QwenVLPlus, os.Getenv("DASHSCOPE_API_KEY"))

    userContent := qwen.VLContentList{
        {Text: "Describe this image in detail"},
        {Image: "https://dashscope.oss-cn-beijing.aliyuncs.com/images/dog_and_girl.jpeg"},
        // local file: {Image: "file:///path/to/image.png"}
    }

    input := dashscopego.VLInput{
        Messages: []dashscopego.VLMessage{
            {Role: "system", Content: &qwen.VLContentList{{Text: "You are a helpful assistant."}}},
            {Role: "user", Content: &userContent},
        },
    }

    streamCallbackFn := func(ctx context.Context, chunk []byte) error {
        fmt.Print(string(chunk))
        return nil
    }

    resp, err := cli.CreateVLCompletion(context.TODO(), &dashscopego.VLRequest{
        Input:       input,
        StreamingFn: streamCallbackFn,
    })
    if err != nil {
        panic(err)
    }

    fmt.Println("\nFull response:", resp.Output.Choices[0].Message.Content.ToString())
}
```

### Audio-Language (Qwen-Audio)

```go
import (
    "context"
    "log"
    "os"

    dashscopego "github.com/the-open-agent/dashscope-go-sdk"
    "github.com/the-open-agent/dashscope-go-sdk/qwen"
)

func main() {
    cli := dashscopego.NewTongyiClient(qwen.QwenAudioTurbo, os.Getenv("DASHSCOPE_API_KEY"))

    userContent := qwen.AudioContentList{
        {Text: "What is the speaker's emotion in this audio?"},
        {Audio: "https://dashscope.oss-cn-beijing.aliyuncs.com/audios/2channel_16K.wav"},
        // local file: {Audio: "file:///path/to/audio.wav"}
    }

    input := dashscopego.AudioInput{
        Messages: []dashscopego.AudioMessage{
            {Role: "system", Content: &qwen.AudioContentList{{Text: "You are a helpful assistant."}}},
            {Role: "user", Content: &userContent},
        },
    }

    resp, err := cli.CreateAudioCompletion(context.TODO(), &dashscopego.AudioRequest{
        Input: input,
        StreamingFn: func(ctx context.Context, chunk []byte) error {
            log.Print(string(chunk))
            return nil
        },
    })
    if err != nil {
        panic(err)
    }

    log.Println("Full response:", resp.Output.Choices[0].Message.Content.ToString())
}
```

### Image Generation (Wanx)

```go
import (
    "context"
    "os"

    dashscopego "github.com/the-open-agent/dashscope-go-sdk"
    "github.com/the-open-agent/dashscope-go-sdk/wanx"
)

func main() {
    cli := dashscopego.NewTongyiClient(wanx.WanxV1, os.Getenv("DASHSCOPE_API_KEY"))

    req := &wanx.ImageSynthesisRequest{
        Model: wanx.WanxV1,
        Input: wanx.ImageSynthesisInput{
            Prompt: "A squirrel painting in the style of Van Gogh",
        },
        Params: wanx.ImageSynthesisParams{
            N: 1,
        },
        Download: true, // download image bytes instead of returning URL
    }

    imgBlobs, err := cli.CreateImageGeneration(context.TODO(), req)
    if err != nil {
        panic(err)
    }

    for _, blob := range imgBlobs {
        // blob.Data contains image bytes when Download: true
        // blob.ImgURL contains the image URL when Download: false
        _ = blob
    }
}
```

### Speech Recognition (Paraformer)

```go
import (
    "bufio"
    "context"
    "fmt"
    "os"
    "time"

    dashscopego "github.com/the-open-agent/dashscope-go-sdk"
    "github.com/the-open-agent/dashscope-go-sdk/paraformer"
)

func main() {
    cli := dashscopego.NewTongyiClient(paraformer.ParaformerRealTimeV1, os.Getenv("DASHSCOPE_API_KEY"))

    req := &paraformer.Request{
        Header: paraformer.ReqHeader{
            Streaming: "duplex",
            TaskID:    paraformer.GenerateTaskID(),
            Action:    "run-task",
        },
        Payload: paraformer.PayloadIn{
            Parameters: paraformer.Parameters{
                SampleRate: 16000, // only 16000 Hz is currently supported
                Format:     "pcm",
            },
            Input:     map[string]interface{}{},
            Task:      "asr",
            TaskGroup: "audio",
            Function:  "recognition",
        },
        StreamingFn: func(ctx context.Context, chunk []byte) error {
            fmt.Print(string(chunk))
            return nil
        },
    }

    // Replace with a real-time audio stream in production
    f, _ := os.Open("/path/to/audio.wav")
    defer f.Close()

    cli.CreateSpeechToTextGeneration(context.TODO(), req, bufio.NewReader(f))
    time.Sleep(5 * time.Second) // wait for final recognition result
}
```

## Layout

```
dashscope-go-sdk/
├── tongyiclient.go        # Main client entry point
├── dtypes.go              # Shared request/response types
├── qwen/                  # Qwen text, VL, and audio models
├── wanx/                  # Wanx image generation
├── paraformer/            # Paraformer real-time ASR
├── embedding/             # Text embedding
├── httpclient/            # HTTP/WebSocket transport layer
└── example/               # Runnable examples
```

## Quick Check

```bash
go test ./...
```

## License

[Apache License 2.0](LICENSE)
