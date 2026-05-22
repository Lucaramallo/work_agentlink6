# Vortex-UI Contributor Response

I'll design these files with clean UX patterns: **pipeline.py** uses Flask for lightweight JSON serving with CORS enabled, while **index.html** implements a card-based layout with real-time fetch metrics and accessible semantic HTML for optimal browser compatibility.

## pipeline.py
```python
from flask import Flask, jsonify
from flask_cors import CORS
import requests
import time

app = Flask(__name__)
CORS(app)

@app.route('/posts', methods=['GET'])
def get_posts():
    start_time = time.time()
    try:
        response = requests.get('https://jsonplaceholder.typicode.com/posts')
        posts = response.json()[:10]
        fetch_time = round((time.time() - start_time) * 1000, 2)
        return jsonify({
            'posts': posts,
            'count': len(posts),
            'fetch_time_ms': fetch_time
        })
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(port=8080, debug=False)
```

## index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Posts Dashboard</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #f5f5f5; padding: 20px; }
        .container { max-width: 800px; margin: 0 auto; }
        header { margin-bottom: 30px; }
        h1 { color: #333; font-size: 28px; margin-bottom: 8px; }
        .metrics { display: flex; gap: 20px; font-size: 14px; color: #666; margin-bottom: 20px; }
        .metric { padding: 8px 12px; background: white; border-radius: 6px; border-left: 3px solid #0066cc; }
        .metric-label { font-weight: 600; color: #333; }
        .posts { display: grid; gap: 16px; }
        .card { background: white; border-radius: 8px; padding: 20px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .card h2 { color: #0066cc; font-size: 18px; margin-bottom: 12px; line-height: 1.4; }
        .card p { color: #555; font-size: 14px; line-height: 1.6; }
        .loading { text-align: center; padding: 40px; color: #999; }
        .error { background: #fee; color: #c33; padding: 16px; border-radius: 6px; margin-bottom: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>Posts Dashboard</h1>
        </header>
        <div class="metrics">
            <div class="metric">
                <span class="metric-label">Records:</span> <span id="count">—</span>
            </div>
            <div class="metric">
                <span class="metric-label">Fetch Time:</span> <span id="time">—</span>
            </div>
        </div>
        <div id="error"></div>
        <div id="content" class="loading">Loading posts...</div>
    </div>

    <script>
        async function fetchPosts() {
            try {
                const response = await fetch('http://localhost:8080/posts');
                const data = await response.json();
                
                if (data.error) throw new Error(data.error);
                
                document.getElementById('count').textContent = data.count;
                document.getElementById('time').textContent = `${data.fetch_time_ms}ms`;
                
                const postsHTML = data.posts.map(post => `
                    <article class="card">
                        <h2>${post.title}</h2>
                        <p>${post.body}</p>
                    </article>
                `).join('');
                
                document.getElementById('content').innerHTML = postsHTML;
            } catch (err) {
                document.getElementById('error').innerHTML = `<strong>Error:</strong> ${err.message}`;
                document.getElementById('content').innerHTML = '';
            }
        }
        
        fetchPosts();
    </script>
</body>
</html>
```

**Setup:** Install Flask (`pip install flask flask-cors requests`), run `python3 pipeline.py`, then open `index.html` directly—no build tools needed.