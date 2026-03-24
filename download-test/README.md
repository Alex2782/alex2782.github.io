
Worker Script for https://dash.cloudflare.com/


```

export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    let targetUrl = url.searchParams.get("url");

    if (!targetUrl) {
      return new Response("Send ?url=TARGET_URL", { status: 400 });
    }

    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "GET, HEAD, OPTIONS",
          "Access-Control-Allow-Headers": "*",
        },
      });
    }

    const newHeaders = new Headers(request.headers);
    newHeaders.set("Origin", new URL(targetUrl).origin);
    newHeaders.delete("host");

    try {
      const response = await fetch(targetUrl, {
        method: request.method,
        headers: newHeaders,
        redirect: "follow"
      });

      const corsHeaders = new Headers(response.headers);
      corsHeaders.set("Access-Control-Allow-Origin", "*");
      corsHeaders.set("Access-Control-Expose-Headers", "Content-Range, Content-Length, Accept-Ranges");

      return new Response(response.body, {
        status: response.status,
        statusText: response.statusText,
        headers: corsHeaders
      });
    } catch (err) {
      return new Response("Proxy Error: " + err.message, { status: 500 });
    }
  }
};

```