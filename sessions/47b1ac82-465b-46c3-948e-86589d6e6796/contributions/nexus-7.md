**Problem:** Need a full-stack solution with a Python backend serving filtered posts and a frontend displaying them with metadata, executable with zero dependencies.

**Solution:** Create a lightweight Flask-free HTTP server in `pipeline.py` using `http.server` that fetches, filters, and serves JSON; pair with vanilla JS `index.html` that fetches from localhost:8080 and renders cards with performance metrics.

**Implementation:**

**pipeline.py:**
```python
import json
import urllib.request
from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse
import time

class PostHandler(BaseHTTPRequestHandler):
    posts_cache = None
    
    def do_GET(self):
        if self.path == '/posts':
            if PostHandler.posts_cache is None:
                try:
                    with urllib.request.urlopen('https://jsonplaceholder.typicode.com/posts') as response:
                        data = json.loads(response.read().decode())
                        PostHandler.posts_cache = data[:10]
                except Exception as e:
                    self.send_response(500)
                    self.end_headers()
                    return
            
            self.send_response(200)
            self.send_header('Content-Type', 'application/json')
            self.send_header('Access-Control-Allow-Origin', '*')
            self.end_headers()
            self.wfile.write(json.dumps(PostHandler.posts_cache).encode())
        else:
            self.send_response(404)
            self.end_headers()
    
    def log_message(self, format, *args):
        pass

if __name__ == '__main__':
    server = HTTPServer(('localhost', 8080), PostHandler)
    print('Server running on http://localhost:8080')
    server.serve_forever()
```

**index.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Posts Viewer</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #f5f5f5; padding: 20px; }
        .container { max-width: 800px; margin: 0 auto; }
        .header { background: white; padding: 20px; border-radius: 8px; margin-bottom: 20px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .stats { font-size: 14px; color: #666; }
        .stats strong { color: #333; }
        .card { background: white; padding: 20px; margin-bottom: 12px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); border-left: 4px solid #007bff; }
        .card h3 { color: #333; margin-bottom: 8px; font-size: 18px; }
        .card p { color: #666; line-height: 1.5; }
        .loading { text-align: center; padding: 40px; color: #666; }
        .error { color: #d32f2f; padding: 20px; background: #ffebee; border-radius: 8px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Posts</h1>
            <div class="stats">
                <span id="count">Records: --</span> | 
                <span id="time">Fetch time: -- ms</span>
            </div>
        </div>
        <div id="posts" class="loading">Loading posts...</div>
    </div>

    <script>
        async function fetchPosts() {
            const start = performance.now();
            try {
                const response = await fetch('http://localhost:8080/posts');
                const posts = await response.json();
                const elapsed = Math.round(performance.now() - start);
                
                document.getElementById('count').textContent = `Records: ${posts.length}`;
                document.getElementById('time').textContent = `Fetch time: ${elapsed} ms`;
                
                document.getElementById('posts').innerHTML = posts.map(post => 
                    `<div class="card">
                        <h3>${post.title}</h3>
                        <p>${post.body}</p>
                    </div>`
                ).join('');
            } catch (error) {
                document.getElementById('posts').innerHTML = 
                    `<div class="error">Error: Ensure pipeline.py is running on http://localhost:8080</div>`;
            }
        }
        
        fetchPosts();
    </script>
</body>
</html>
```

**Execution:** Run `python3 pipeline.py`, open `index.html` directly in browser—zero dependencies, CORS-enabled, sub-second load.