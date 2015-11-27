General Notes
	Node doesn't check whether Content-Length === the length of the body

	If utilizing sockets, check the 'clientError' event for error handling/logging (look below)

HTTP server/client
	require('http')

	The module never buffers entire requests/responses
		Thus, the user is able to stream data

	Very low level
		Module parses HTTP messages into headers and body but that's it
			Allows for many implementations

	http.Agent class
		Used for pooling sockets used in HTTP client requests
			Pool
				Keeps a certain number of sockets to be used by the program
					The sockets reused for different requests, so having a pool reduces the overhead of creating new sockets.  This also makes it so there's a limit on the number of sockets utilized
				Keeps a queue of requests, that will be offloaded onto sockets when the socket becomes available.

				The true maximum number of simultaneous sockets is controlled by the OS, but you can alter the options.maxSockets in node (which can't be greater than the OS number)

				There is a global pool that can be utilized, but it will not give granular control.

		new Agent([options])
			options OBJ
				keepAlive BOOL
					(DEFAULT-False)- Keep sockets around in a pool to be used by other requests in the future
				
				keepAliveMsecs INT DEF-1000
					When keepAlive is true, this setting dictates how often to send TCP KeepAlive packets over sockets that are being kept alive.
				
				maxSockets INT DEF-INFINITY
					Max number of sockets to allow per host
				
				maxFreeSockets INT DEF-256
					PREREQ: keepAlive == true
					Max number of sockets to leave open in a free state.

			The default http.globalAgent that's used by http.request has these values set to their respective defaults.
				If you want to customize, you must create your own Agent object.

				EX:
				var http = require('http');
				var keepAliveAgent = new http.Agent({ keepAlive: true });
				options.agent = keepAliveAgent;
				http.request(options, onResponseCallback);

			agent.destroy()
				Destroy any sockets that are in use by the agent

				You usually don't have to do this, but if you're using an agent with KeepAlive == true, it's best to utilize this function to explicitly shut down the agent that you're no longer using.

			agent.freeSockets OBJ
				PREREQ: keepAlive == true
				Contains arrays of sockets currently awaiting use by the Agent.

				Do NOT modify this OBJ

			agent.getName(options)
				For a set of request options, get a unique name to determine whether a connection can be reused.

				In the http agent, this returns
					host:port:localAddress
				In the https agent, this returns
					CA, cert, ciphers, and other HTTPS/TLS-specific options that determine the reusability.

			agent.maxFreeSockets INT
			agent.maxSockets INT
				Another way to set these configs (see above)

			agent.requests OBJ
				Contains queues of requests that have not yet been assigned to sockets.

				Don't modify this

			agent.sockets OBJ
				An object which contains arrays of sockets currently in use by the Agent

				Don't modify this

	http.ClientRequest
		The object that's returned from http.request()

		Is a Writable Stream and is an EventEmitter

			This represents an in-progress request whose header has already been queued.
				At this point, the header is still mutable using the header API in Node

				The actual header will be sent along with the first data chunk or when closing the connection.

			To get the response, add a listener for 'response' to the request object.
		
		events
			The request is an EventEmitter

			response
				args
					instance of http.IncomingMessage

				add a listener for this event to the request object.

				emitted when the response headers have been received

				If you add a response event handler, you MUST consume the data from the response object.  
					You can do this by:
						response.read()
							Whenever there's a readable event
						adding a 'data' handler
						calling .resume()
					
					Until the data is consumed, the 'end' event will not fire.  Additionally, until the data is read it will consume memory that can lead to errors.

			abort
				The request has been aborted by the client.

				Only emitted on the first call to abort()

			connect
				
				Emitted each time a client requests a http CONNECT method.
					CONNECT is an HTTP verb that is used for tunneling purposes.
						Ex: If we open a telnet session and DO
							CONNECT zachroof.com:443

							// The proxy will answer with 
							HTTP/1.0 200 Connection established

							This means that the proxy set up the tunnel to the host.  Now you can communicate with the host through the tunnel.


				function (response, socket, head) { }

				EX:
					//if confused on the ordering of events, look at http://stackoverflow.com/questions/25941175/calling-nodejs-request-end-method-before-setting-up-the-listeners

				var http = require('http');
				var net = require('net');
				var url = require('url');

				// Create an HTTP tunneling proxy
				var proxy = http.createServer(function (req, res) {
				  res.writeHead(200, {'Content-Type': 'text/plain'});
				  res.end('okay');
				});
				proxy.on('connect', function(req, cltSocket, head) {
				  // connect to an origin server
				  var srvUrl = url.parse('http://' + req.url);
				  var srvSocket = net.connect(srvUrl.port, srvUrl.hostname, function() {
				    cltSocket.write('HTTP/1.1 200 Connection Established\r\n' +
				                    'Proxy-agent: Node.js-Proxy\r\n' +
				                    '\r\n');
				    srvSocket.write(head);
				    srvSocket.pipe(cltSocket);
				    cltSocket.pipe(srvSocket);
				  });
				});

				// now that proxy is running
				proxy.listen(1337, '127.0.0.1', function() {

				  // make a request to a tunneling proxy
				  var options = {
				    port: 1337,
				    hostname: '127.0.0.1',
				    method: 'CONNECT',
				    path: 'www.google.com:80'
				  };

				  var req = http.request(options);
				  req.end();

				  req.on('connect', function(res, socket, head) {
				    console.log('got connected!');

				    // make a request over an HTTP tunnel
				    socket.write('GET / HTTP/1.1\r\n' +
				                 'Host: www.google.com:80\r\n' +
				                 'Connection: close\r\n' +
				                 '\r\n');
				    socket.on('data', function(chunk) {
				      console.log(chunk.toString());
				    });
				    socket.on('end', function() {
				      proxy.close();
				    });
				  });
				});

		continue
			emitted when the server sends a '100 Continue' HTTP response.

			This is an instruction that the client should send the request body.

		response
			function (response) { }

			Emitted when a response is received to this request.

			Emitted only once

			argument is an instance of http.IncomingMessage

		socket
			Emitted after a socket is assigned to this request

		upgrade
			function (response, socket, head) { }

			Emitted each time a server responds to a request with an upgrade

			EX: A client server pair that show you how to listen for the upgrade event.

			var http = require('http');

			// Create an HTTP server
			var srv = http.createServer(function (req, res) {
			  res.writeHead(200, {'Content-Type': 'text/plain'});
			  res.end('okay');
			});
			srv.on('upgrade', function(req, socket, head) {
			  socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
			               'Upgrade: WebSocket\r\n' +
			               'Connection: Upgrade\r\n' +
			               '\r\n');

			  socket.pipe(socket); // echo back
			});

			// now that server is running
			srv.listen(1337, '127.0.0.1', function() {

			  // make a request
			  var options = {
			    port: 1337,
			    hostname: '127.0.0.1',
			    headers: {
			      'Connection': 'Upgrade',
			      'Upgrade': 'websocket'
			    }
			  };

			  var req = http.request(options);
			  req.end();

			  req.on('upgrade', function(res, socket, upgradeHead) {
			    console.log('got upgraded!');
			    socket.end();
			    process.exit(0);
			  });
			});

	request.abort()
		Marks the request as aborting.

		This will cause remaining data in the response to be dropped and the socket will be destroyed

	request.end([data][, encoding][, callback])
		Finishes sending the request

		If any parts of the body are unsent, it will flush them to the stream.

		If the request is chunked, this will send a string of terminating characters (aka blank line or '0\r\n\r\n')

	request.setNoDelay([noDelay])
		Once a socket is assigned to this request and is connected socket.setNoDelay() will be called.

	request.setSocketKeepAlive([enable][, initialDelay])#
		Once a socket is assigned to this request and is connected socket.setKeepAlive() will be called.

	request.setTimeout(timeout[, callback])#
		Once a socket is assigned to this request and is connected socket.setTimeout() will be called.

	request.write(chunk[, encoding][, callback])
		Sends a chunk of the message BODY

		By calling this many teams, a user can stream to the server.
			Suggested to use ['Transfer-Encoding', 'chunked'] header line when creating the request

		args
			chunk
				Can be a Buffer or string

			encoding
				Only use when the chunk is a string

			callback
				called once the chunk of data has been flushed to the underlying resource (aka filesystem)







		Sockets are removed from the agent's pool when the socket emits either a "close" event or a special "agentRemove" event.

			Ex: If you want to keep one HTTP request open for a long time and don't want it to stay in the pool

			http.get(options, function(res) {
			  // Do stuff
			}).on("socket", function (socket) {
			  socket.emit("agentRemove");
			});

			//Or, you could just opt out of pooling entirely using this..

			http.get({
			  hostname: 'localhost',
			  port: 80,
			  path: '/',
			  agent: false  // create a new agent just for this one request
			}, function (res) {
			  // Do stuff with response
			})



		Defaults client requests to Connection:keep-alive
			The connection to the server will be persisted until the next request (or an attempt at this is made)
			If no pending HTTP requests are waiting on a socket to become free the socket is closed
				This means Nodes pool has the benefit of keep-alive when under load but still doesn't require devs to manually close the HTTP clients using KeepAlive.
					You can opt into using HTTP KeepAlive (see below)

			Agent option object
				KeepAlive
					set to true
						The Agent will keep unused sockets in a pool for later use.
							These sockets will be marked so the Node process wont be kept running indefinitely (due to references to sockets in the event loop)
					Best practice to explicitly destroy() KeepAlive agents when they aren't in use, so that Sockets will be shut down.




		Ex:
		pool = new http.Agent(); //Your pool/agent
		http.request({hostname:'localhost', port:80, path:'/', agent:pool});
		request({url:"http://www.google.com", pool:pool });

	http.Server
		EventEmitter with the following events

		Event
			'clientError'
				function (exception, socket) { }

				If a client connection emits an 'error' event, it will be forwarded here.

				socket is the net.Socket object that the error originated from

			close
				Emitted when the server closes

			connect
				function (request, socket, head) { }

				Emitted each time the client requests a http CONNECT method

				args
					request
						args for the HTTP request

					socket
						network socket between the client and server

					head
						an instance of Buffer, the first packet of the tunneling stream (which might be empty)

