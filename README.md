<h1 class="card-title">HTTP Web Server Front-end</h1>

Run the following in your local "webroot" folder using Claude Code CLI to start an http server on port 8887:

	start server

The "start server" command runs ./desktop/install/quickstart.sh which creates a virtual environment in desktop/install/env/ if it doesn't exist, and starts the Python HTTP server with server-side execution access.

Or run a basic http server without server-side execution:

	python -m http.server 8887

Then view pages at:

[localhost:8887/](http://localhost:8887/)  
[localhost:8887/team](http://localhost:8887/team/)  
[localhost:8887/comparison](http://localhost:8887/comparison/)  
[localhost:8887/realitystream](http://localhost:8887/realitystream/)  
[localhost:8887/localsite](http://localhost:8887/localsite/)  
[localhost:8887/home](http://localhost:8887/home/)  
[localhost:8887/feed](http://localhost:8887/feed/)  



Look at the code in your webroot with an editor like [Sublime Text](https://www.sublimetext.com/) ($99), [VS Code](https://code.visualstudio.com/) or [WebStorm](https://www.jetbrains.com/webstorm/).