# demo-repo
Dự án được quản lý bởi AI agent
Dự án này được quản lý với sự hỗ trợ của AI Agent
GitHub Agent đã tích hợp thành công vào khung chat chính

## Hướng dẫn sử dụng Fetch API

Trong Cloudflare Workers, `fetch()` là API chuẩn để gọi HTTP request đến các API bên ngoài (hoặc đến chính Worker khác). Workers hỗ trợ đầy đủ Fetch API theo chuẩn web, với một số lưu ý riêng của môi trường serverless.

### Gọi fetch đến API bên ngoài (async/await)

```js
export default {
  async fetch(request, env, ctx) {
    try {
      const response = await fetch('https://api.example.com/data', {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          // 'Authorization': `Bearer ${env.API_TOKEN}`,
        },
      });

      if (!response.ok) {
        return new Response(
          JSON.stringify({ error: `Upstream trả về lỗi: ${response.status}` }),
          { status: response.status, headers: { 'Content-Type': 'application/json' } }
        );
      }

      const data = await response.json();
      return new Response(JSON.stringify(data), {
        headers: { 'Content-Type': 'application/json' },
      });
    } catch (err) {
      return new Response(
        JSON.stringify({ error: 'Không thể kết nối đến API bên ngoài', detail: String(err) }),
        { status: 502, headers: { 'Content-Type': 'application/json' } }
      );
    }
  },
};
```

### Xử lý JSON

- Đọc body JSON từ response: `const data = await response.json();`
- Gửi JSON trong request: đặt header `Content-Type: application/json` và `body: JSON.stringify(payload)`.
- Luôn kiểm tra `response.ok` (hoặc `response.status`) trước khi parse body.

### Xử lý lỗi

- Bọc `fetch` trong `try/catch` để bắt lỗi mạng/timeout.
- Kiểm tra `response.ok`; nếu không ok, đọc `response.status` và nội dung lỗi từ upstream.
- Trả về mã lỗi phù hợp cho client (ví dụ 502 khi upstream lỗi).
- Có thể dùng `AbortSignal.timeout(ms)` để giới hạn thời gian chờ.

### Tài liệu chính thức

- Cloudflare Workers — Fetch API: https://developers.cloudflare.com/workers/runtime-apis/fetch/
