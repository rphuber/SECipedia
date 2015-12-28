The CGI system also allows the Web browser to send information to the script via the URL or an HTTP POST request. If a slash and additional directory name(s) are appended to the URL immediately after the name of the script, then that path is stored in the PATH_INFO environment variable before the script is called. If parameters are sent to the script via an HTTP GET request (a question mark appended to the URL, followed by param=value pairs), then those parameters are stored in the QUERY_STRING environment variable before the script is called. If parameters are sent to the script via an HTTP POST request, they are passed to the script's standard input. The script can then read these environment variables and adapt to the Web browser's request. --Wikipedia
  Think about directory traversal hacks or other items by manipulating the params sent into the script.  Think of SQL injection type logic here...