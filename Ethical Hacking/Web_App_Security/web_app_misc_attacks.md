
Changing the price of a product that is transmitted in a hidden HTML form field to fraudulently purchase the product for a cheap amount.

If you compromise a user's email account, you have access to everything.

Access Control Flaws
  Often the developer makes wrong assumptions about how a user will interact with the application and will omit access control checks from some application functions.

  To find these, it is tedious.  You need to do probing on every independent part of the application.

Tokenizing Flaws
  Usually created when trying to blacklist

  exs:
    SELECT/*foo*/username,password/*foo*/FROM/*foo*/users
      Utilizing comment syntax


    <img%09onerror=alert(1) src=a>
      Using encoded characters


  Black list filters in WAFs
    Vulnerable to NULL byte attacks
      Inserting a NULL byte anywhere before a blocked expression can cause some filters to stop processing the input and therefore not identify the expression.
        Ex: %00<script>alert('muhah')</script>

      These types of attacks are relatively common in the web app security world.
        As the NULL byte can act as a string delimiter (in certain contexts), it can be used to terminate a filename or a query to some back-end component.

    In some browsers, NULL bytes are simply ignored; so arbitrary null bytes can be inserted with blocked expressions to defeat certain blacklist-based filters.

When crafting input for an attack, think of the ways that the data can be potentially transformed before any white list/sanitizing is met.
  Ex: Is recursive sanitizing in place?
    If not: <scr<script>ipt> becomes <script>

  Ex: If the application first removes ../ recursively and then removes ..\ recursively then the following input can be used to defeat the validation
      ....\/

      If recurive sanitization occurs, when if one one problematic character leads to another.  There could be an recurive loop that crashes the server.

In many apps, admin functions are implemented within the app itself (and is accessible through the same web interface as its core functionality)
      Thus greatly opens up the attack surface.
        Often times, apps dont implement effective access control of some of their admin functions.  Thus, can attacker may find a means of creating a new user account with powerful privs.

        Admin functions usually involves displaying data that originated from ordinary users.  Any XSS flaws within that admin interface can be a huge point of concern.

        Admin functionality is often subjected to less stringent security testing because its users are "trusted" or pen-testers are given access to only low-privileged accounts.

HTTP Headers
  Expires, Pragma headers
    Are leveraged to tell the browser to cache the response.  If you're doing an attack, you can always instruct the clients to cache the contents so when the website changes it's code, it doesn't really matter (unless the cache is flushed).

  Set-Cookie
    Can domains from one domain interfere with cookies for another? What if they are set "globally"?  Would a SOP still apply to the cookie? 


  Refer
      If sensitive information is kept in the URL, it can be easily seen in the referer header if a link is followed.
          What about a person on a website with a sensitive URL and then they click on an ad link, this information can be leaked to the ad's server.  
              Implications for HTTPS? I believe that it is obfuscated from view in this context.




