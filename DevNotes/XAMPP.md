#### Setup Apache to serve on custom host (domain) instead of localhost
```markdown
- Run XAMPP, and start Apache server.
- Press the `Config` button and choose `<Browse Apache>`.
- Inside the just opened directory to `conf` directory and open `httpd.conf` configuration file.
- Inside this document find the `Vritual hosts` subsection. It is located under `Supplemental configuration` section. And usually consist of line like this `Include conf/extra/httpd-vhosts.conf`, which is a directive to include a supplementary configuration file into main configuration file.
- Take the path after the `conf/` and navigate to this file (i.e. extra/httpd-vhosts.conf).
- This file should contain some instructions and commented out examples.
- Copy one of this exapmles, uncomment opening and closing tags and add values for `DocumentRoot` and `ServerName` keys. Like this
<VirtualHost *:80>
    DocumentRoot "C:/xampp/htdocs/phpiggy/public"
    ServerName phpiggy.local
</VirtualHost>
* DocumentRoot - it's root path where Apache will search for files to serve. By default it will serve the index.php file under this directory.
* ServerName - is the domain name that the apache will respond to. It may be anything you want, but it's good practice to add the `.local` to denote this is a local project.
- Stop and start the Apache service via XAMPP.

At this moment we have setup Apache to serve requests for `phpiggy.local` domain.
But if we make request to `phpiggy.local` via our browser, the browser would attemp to search for `phpiggy.local` domain on the internet.

To prevent browser from doing so, we can manipulate the `hosts` file. This file is the first thing a browser looks for when trying to resolve the domain name. Usually it contains comments and examples. In this file we can map our domain name to localhost.

- Run NotePad as admin. Open `C:\Windows\System32\drivers\etc\hosts`. If you can't see `hosts` file, change file type to `All`.
- At the end of the document add a comment that describes what you have added and your mapping. Like this:
 # XAMPP Virtual Hosts
 127.0.0.1 phpiggy.local

Now if you enter `http://phpiggy.local`  in the browser search bar it would output the contents if the index.php file located in the directory from `DocumentRoot` directive in the Apache virtual hosts config.
```
---
