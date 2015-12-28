Security Tips In Apache
  Always stay updated!
    Subscribe to the main mailing list for various posts on server upgrades and security issues (important issues only)
    Not a discussion thread, only messages are posted through the foundation
        announce-subscribe@httpd.apache.org

        announce-help@httpd.apache.org
          To get help with the list

    Third party code
      Most of the time, if there's a vulnerability, it isn't in the Apache Server code.  It's from add-on code, CGI scripts, or the OS.
        You must keep abreast of updates/vulnerabilities to these items.


  Denial of Service (DoS) Attacks
    Basic Idea
      Application servers typically allocate an operating system thread/process for each connection.  As a process is a relatively memory intensive resource, it's very easy to initiate DoS attacks.

      "HTTP Heavy Lifting"

    Try to prevent responses to clients by typing up resources on the server.

    You can't prevent these attacks entirely, but you can do create mitigating factors.

    Create a multi-tier solution 
      In addition to the security tips outlined here, it's very important to have correct firewall configurations
        We will look at this in depth in another lecture.

        Many firewalls can be configured to restrict the number of simultaneous connections from any unique IP/network.
          This isn't effective against DDoS attacks (Distributed Denial of Service Attacks)

    Server Configs that can help if you're receiving a DoS attack (or are highly vulnerable to them)
      
      KeepAliveTimeout directive
        Number of seconds apache will wait before closing the connection.
          Thus, if no requests are sent in a certain time frame, the connection is "idle" and it is closed.

          Once the connection is established, the timeout value specified in the Timeout directive applies.
            It is important to know the differences between the directives.

        KeepAlive (On|Off) Directive
          You can turn KeepAlive on/off as well

          However, just like lowering the KeepAliveTimeout, there can be performance implications to this action.
            
            Without KeepAlive
              For every client request 
                  
                  Opening connection
                    Three-way TCP handshake
                    
                  Client sends an HTTP request
                  Client receives an HTTP response

                  Closing connection
                    Two-way handshake

          In a DoS attack, you must weigh the costs/benefits when changing keepAlives
            Pros: Concurrency portion of DoS in mitigated
            Cons: Extra overhead of connections being established







      
      RequestReadTimeout directive
        Sets a limit for the amount of time a client can take when finishing the request

      TimeOut Directive
        If receiving a DoS attack, one should look at this config first.
          Ex: The default is 1 minute, but if you're receiving a DoS attack, it might be wise to lower this to a few seconds.

        Be very careful, as this is an "aggregate" directive which affects multiple settings that CAN have adverse consequences.

        Has a larger footprint then the RequestReadTimeout directive.
          This directive sets a timeout for
            reading data from a client
              But what if a client trickles data very slowly?
                Hard to mitigate against this.

            When sending data to a client, the maximum time to wait for a acknowledgment of a packet (ACK) when the send buffer is full (client is "slow" to receive packets)

            Max waiting time for output from a CGI script
              If you have running CGI scripts, you need to make sure that your timeout isn't too low!  This might effect this setting!

            Max waiting time for output from a filtering process
              Filtering processes can include compression, etc.
                Make sure that your timeout value takes into account filters that are in place already!

            If you're utilizing Apache as a Proxy and have NOT set the ProxyTimeout directive
              This will set a timeout on proxy processes.
                See the mod_proxy documentation (anything the references the default timeout value) for the scope of the timeout.



