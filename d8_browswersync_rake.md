# Drupal 8 + Browsersync + Rake

Manually refreshing your page and  clearing the cache, aka`drush cr`, are two things that will slow your Drupal development time. What if your browser reloaded automatically after changes, and across multiple devices and browsers you have open? This is where Browsersync comes in. Browsersync is module for Node.js that allows you to sync your changes across browsers and devices.

# Preparing Drupal
This article assumes you have a working install of Drupal 8 and a theme in place. If you don’t, check out Joe Komenda’s post, [Up and Theming with Drupal 8](https://thinkshout.com/blog/2015/11/up-and-theming-with-drupal-8/). This will get you going.

Once you have D8 installed you’ll need to turn off caching. Rename `sites/example.settings.local.php` to `sites/example.settings.local.php`.  You can rename the files from your editor of choice, if you prefer, or run the following command from your site root :

~~~ shell
$ cp sites/example.settings.local.php sites/default/settings.local.php
~~~

To be sure your changes are included we’ll need to enable Drupal’s Null Cache Service. Uncomment the following line `sites/default/settings.php`:

~~~ php
$settings['container_yamls'][] = DRUPAL_ROOT . '/sites/development.services.yml';
~~~

Next, let’s disable the render cache and dynamic page cache. Uncomment the following in the same file.

~~~ php
$settings['cache']['bins']['render'] = 'cache.backend.null';
$settings['cache']['bins']['dynamic_page_cache'] = 'cache.backend.null';
~~~

Finally, add the following to `sites/development.service.yml`:
~~~ yaml
parameters:
  twig.config:
    debug : true
    auto_reload: true
    cache: false
~~~

Run `drush cr` from the root of your site to rebuild the cache.

# Installing Browsersync
Browser sync is installed using Node Package Manager (NPM). If you already have Node.js, then you already have NPM. If you don’t have it installed, head over to [nodejs.org](https://nodejs.org/en/).

Once Node.js and NPM are set up, install Browsersync with `npm install -g browser-sync`.  This will install it globally so that you don’t have to reinstall it every time you spin up a new project. Test that your installation is working by running `browser-sync -h` in your terminal. That should show all the usage, commands and options for the plugin.

# Connecting Browsersync to Drupal
Let’s make the magic happen by connecting Drupal and Browsersync. Go to the root of your Drupal theme folder. Run `browser-sync start`. Browsersync will generate a script tag for you to place just before the closing body tag. Browser sync also has  UI. You’ll see a URL for your localhost and one for sharing the connection to other devices on the same network.

![Browsersync start](https://raw.githubusercontent.com/heypaxton/Posts/master/img/browsersync-start.png)

Let's add the script tag to your `html.html.twig`. 

~~~ html
<!DOCTYPE html>
<html{{ html_attributes }}>
  <head>
    <head-placeholder token="{{ placeholder_token|raw }}">
    <title>{{ head_title|safe_join(' | ') }}</title>
    <css-placeholder token="{{ placeholder_token|raw }}">
    <js-placeholder token="{{ placeholder_token|raw }}">
  </head>
  <body{{ attributes.addClass(body_classes) }}>
    <a href="#main-content" class="visually-hidden focusable skip-link">
      {{ 'Skip to main content'|t }}
    </a>
    {{ page_top }}
    {{ page }}
    {{ page_bottom }}
    <js-bottom-placeholder token="{{ placeholder_token|raw }}">
    <script id="__bs_script__">
      //<![CDATA[
        document.write("<script async src='http://HOST:3000/browser-sync/browser-sync-client.2.11.2.js'><\/script>".replace("HOST", location.hostname));
      //]]></script>
    </script>
  </body>
</html>
~~~

# Task management with Rake
