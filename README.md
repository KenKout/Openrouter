# FastAPI 

## Mô tả

Bạn sẽ xây dựng một proxy API bằng **FastAPI** để chuyển tiếp các yêu cầu đến **OpenRouter API**. Tuy nhiên, OpenRouter có một vấn đề: nó không trả về **status code chính xác** khi có lỗi, điều này có thể gây nhầm lẫn cho client.

Nhiệm vụ của bạn là xử lý phản hồi từ OpenRouter sao cho phù hợp với format của **OpenAI API**, đảm bảo rằng:

- Khi `stream == false`, API của bạn sẽ trả về **status code chính xác** dựa trên nội dung phản hồi từ OpenRouter.
- Khi `stream == true`:
  - Nếu OpenRouter **không có lỗi**, bạn sẽ **stream dữ liệu** như bình thường.
  - Nếu OpenRouter **trả về lỗi**, bạn sẽ **trả JSONResponse với status code phù hợp**, thay vì stream dữ liệu lỗi.

## API Endpoint

Bạn cần tạo một **POST endpoint** `/proxy` có chức năng gửi request đến OpenRouter API tại:

```
POST https://openrouter.ai/api/v1/chat/completions
```

### Ví dụ request

```json
{
  "model": "qwen/qwen2.5-vl-32b-instruct:free",
  "messages": [
    {
      "role": "user",
      "content": "Hi"
    }
  ],
  "stream": true
}
```

### Vấn đề cần giải quyết

#### Trường hợp `stream == false`
- OpenRouter trả về **status code 200** nhưng trong body có lỗi:
  ```json
  {
    "error": {
      "message": "Rate limit exceeded: free-models-per-day",
      "code": 429
    }
  }
  ```
- Bạn cần **trả về response với status code 429** thay vì 200.

#### Trường hợp `stream == true`
- OpenRouter trả về **text/event-stream** nhưng vẫn có lỗi trong nội dung:
  ```
  : OPENROUTER PROCESSING
  
  data: {"error":{"message":"Rate limit exceeded: free-models-per-day","code":429}}
  ```
- Bạn cần **phát hiện lỗi trong stream** và thay vì trả `content-type == text/event-stream`, hãy trả về JSON response với status code phù hợp.

## Yêu cầu hoàn thành

- Viết một **FastAPI router** có endpoint `/proxy`.
- Xử lý phản hồi từ OpenRouter để đảm bảo API của bạn hoạt động theo format của OpenAI.
- Đảm bảo rằng khi có lỗi, API của bạn trả về **status code chính xác**.
- Viết code **sạch sẽ, dễ hiểu, có chú thích đầy đủ**.

## Gợi ý

- https://stackoverflow.com/questions/77795766/aiohttp-connection-closing-before-fastapi-streaming-response
- https://stackoverflow.com/questions/73721736/what-is-the-proper-way-to-make-downstream-http-requests-inside-of-uvicorn-fastap/73736138#73736138
