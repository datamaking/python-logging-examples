To enable HTTP API logs in a Python Flask application, you can use Flask's built-in logging capabilities or integrate middleware to capture request/response details. Here's a step-by-step guide:

---

### **1. Basic Logging with Flask's Built-in Logger**
Flask uses Python's `logging` module by default. Configure it to log HTTP requests and responses:

```python
from flask import Flask, request
import logging

app = Flask(__name__)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

@app.route('/')
def home():
    app.logger.info(f"Request received: {request.method} {request.path}")
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

**Output**:
```
2023-10-05 12:34:56,789 - INFO - Request received: GET /
```

---

### **2. Log All HTTP Requests with Middleware**
Use Flask's `before_request` and `after_request` hooks to log every request/response:

```python
from flask import Flask, request
import logging
import time

app = Flask(__name__)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

@app.before_request
def log_request():
    request.start_time = time.time()

@app.after_request
def log_response(response):
    # Calculate request duration
    duration = time.time() - request.start_time

    # Log request details
    app.logger.info(
        f"Method: {request.method} | "
        f"Path: {request.path} | "
        f"Status: {response.status_code} | "
        f"Duration: {duration:.2f}s"
    )
    return response

@app.route('/')
def home():
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

**Output**:
```
2023-10-05 12:34:56,789 - INFO - Method: GET | Path: / | Status: 200 | Duration: 0.02s
```

---

### **3. Advanced Logging with `Werkzeug`**
Flask's underlying server (`Werkzeug`) has its own logger. Enable it for detailed request logs:
```python
from flask import Flask
import logging

app = Flask(__name__)

# Enable Werkzeug's logger
werkzeug_logger = logging.getLogger('werkzeug')
werkzeug_logger.setLevel(logging.INFO)

@app.route('/')
def home():
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

**Output** (Werkzeug logs):
```
127.0.0.1 - - [05/Oct/2023 12:34:56] "GET / HTTP/1.1" 200 -
```

---

### **4. Log to a File**
Save logs to a file using `RotatingFileHandler`:
```python
from flask import Flask
import logging
from logging.handlers import RotatingFileHandler

app = Flask(__name__)

# Configure file logging
handler = RotatingFileHandler(
    'app.log',
    maxBytes=10000,  # 10KB per file
    backupCount=3  # Keep 3 backup files
)
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
app.logger.addHandler(handler)

@app.route('/')
def home():
    app.logger.info("Request received!")
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

---

### **5. Use Structured Logging (JSON)**
For production, log in JSON format using libraries like `python-json-logger`:
```bash
pip install python-json-logger
```

```python
from flask import Flask, request
import logging
from pythonjsonlogger import jsonlogger

app = Flask(__name__)

# Configure JSON logging
handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter(
    '%(asctime)s %(levelname)s %(message)s %(pathname)s %(funcName)s'
)
handler.setFormatter(formatter)
app.logger.addHandler(handler)
app.logger.setLevel(logging.INFO)

@app.route('/')
def home():
    app.logger.info("Request received", extra={
        'method': request.method,
        'path': request.path,
        'ip': request.remote_addr
    })
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

**Output** (JSON):
```json
{
  "asctime": "2023-10-05 12:34:56,789",
  "levelname": "INFO",
  "message": "Request received",
  "method": "GET",
  "path": "/",
  "ip": "127.0.0.1"
}
```

---

### **Key Tips**
- **Filter Sensitive Data**: Avoid logging headers like `Authorization` or request bodies with passwords.
- **Use Gunicorn in Production**: If deploying with Gunicorn, configure its logging instead:
  ```bash
  gunicorn --access-logfile - --error-logfile - app:app
  ```
- **Log Rotation**: Use `RotatingFileHandler` or tools like `logrotate` to manage log files.

Start with basic logging and expand based on your needs!
