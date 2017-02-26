---
layout:     post
title:      "Hardening WordPress"
subtitle:   "How to secure your WordPress site?"
date:       2016-12-04 0:00:00
author:     "W3ndige"
header-img: "img/hardening-wordpress-header.png"
category: "Web Security"
---

<h1>Introduction</h1>

<p>WordPress is the most popular content management system (27.2% of all websites in 2016), bringing an ease in creating websites even for non technical users. Perfect option? Unfortunately not, as WordPress is also the most hacked platform (around 70% of all hacked CMS are WordPress based sites). </p>

<p>Is it because it's so popular? Maybe because many people don't have basic technical knowledge allowing to secure these platforms? Or because people create these websites and then forget about updating? I leave it to your answer. </p>

<p>Today we're going to focus on ways to secure your site, making it harder to get hacked. </p>

<h1>Keep your WordPress up to date</h1>

<p>First and the most important thing is to always keep your site updated, as well as plugins and themes that you're using as developers are working hard to secure all the reported flaws. A lot of sites were hacked by vulnerabilities in code of old plugins, or WordPress core, which were updated in the newer versions. </p>

<p>If you don't want to come back to your website as often, to only check for updates, consider auto update option. You can do this by using one of two methods - by defining constant in wp-config.php or by adding plugins. </p>

<b>wp-config.php</b>

<p>To enable automatic updates define this constant: </p>

{% highlight text %}
define( 'AUTOMATIC_UPDATER_DISABLED', true );
{% endhighlight %}

<ul>
<li>True – Development, minor, and major updates are all enabled</li>
<li>False – Development, minor, and major updates are all disabled</li>
<li>'minor' – Minor updates are enabled, development, and major updates are disabled</li>
</ul>
<p>For plugins: </p>

{% highlight text %}
add_filter( 'auto_update_plugin', '__return_true' );
{% endhighlight %}

<p>For themes: </p>

{% highlight text %}
add_filter( 'auto_update_theme', '__return_true' );
{% endhighlight %}

<h1>Use strong credentials!</h1>
<ul>
<li><b>Strong passwords.</b></li>
<li><b>Don't use admin as username.</b></li>
</ul>

<p>Not only people will try to attack your site. There are bots that will try to access your <i>wp-admin</i> page and brute force your password, preferably using <i>admin</i> as username. So it's essential to have strong password, but also by changing the username to something less obvious. That way attacker will also have to find it, along with the password.  </p>

<p>For passwords, I can reccomend generators from apps like KeePass or LastPass. Your password have to be <b>CLU</b>: Complex. Long. Unique.</p>

<p>Another security practice is having two accounts - one working as admin and second for publication. Publication account should be used while adding blog posts, content that don't require admin privilages, and while using some public internet access. That way, even if somehow attacker gained access to your password, privilages of publication account won't allow him to do much more harm, than while being logged as admin. </p>

<p>Also - <b>consider two factor authentication!</b></p>

<h1>Think about your plugins!</h1>

<p>If you're not using a plugin, it's deactivated - simply, delete it. It's known to attack a site, by exploiting flaws in the plugin that the developer wasn't even using. But why it happens? After deactivating the plugin - its file are still in the WordPress directory waiting to be used once again. Or maybe waiting to be hacked? </p>

<p>Always download plugins from trustworthy sources. Even if you want a premium plugin, download them from sites like <a href="https://themeforest.net/">Themeforest</a>, or from developer's website. Those free versions may be infected with malware or some backdoor that will make your site very easy to exploit. </p>

<h1>Hide your wp-config.php and .htaccess files</h1>
<p>The wp-config file contains very confidentional information about your website. To protect it from intrusion, you should add the following code to your .htaccess file to deny access to anyone else. </p>

{% highlight text %}
<Files wp-config.php>;
order allow,deny
deny from all
</Files>
{% endhighlight %}
<p>Same for .htaccess file.</p>

{% highlight text %}
<Files .htaccess>
order allow,deny
deny from all
</Files>
{% endhighlight %}

<h1>Change your WordPress Security Keys</h1>

<pre>
define('AUTH_KEY',         'VatTc3WfW~dKdE~m=Hf_bvaH-)1+ko=IQ:!oiYW}o1H+[;LgP[QC981U+{K2w0d+');
define('SECURE_AUTH_KEY',  '/@*MX4vaG(:8iMfx(V z$-4b<N;_((cA `nv/~,*tKiO6W}hUf+ t]rL[I9,m^o|');
define('LOGGED_IN_KEY',    'c$Ou*1`?^+(:I4=Lpy6aD-HMl/1Jdg9y-J$1^(V+o>{TyE[q.$)Rtvm:q5j_itG.');
define('NONCE_KEY',        '=+2:9RPE=0x`9!/l}qaYQ`{m)OW|s2I&Hnk=O4/;gwh}I+WGJCk2PmHK{1CIjhDm');
define('AUTH_SALT',        '$5a?Qk1Rzfd{+r_/9XR%2<P+f1^QlQ|pJXMX>1ly-8<n%1J#IaE+O9xy=8(x/AWU');
define('SECURE_AUTH_SALT', 'iehUhOYms(1fQzpacQTrfUR?s;g>9-?(Q=H0s:p!7pT@xmAu/o>90MI2NE1-z-ji');
define('LOGGED_IN_SALT',   '$.[D]<Nm)N[gig#<Dc8|>FXr&+QMmN{sv(AgHM.=*Fdnz-)1YTBAOuJIr94w0Mo3');
define('NONCE_SALT',       'kkv<g?I-JZ+J%xdS!~9vr>F4myL7hdgl++7Q|)I|,=^Jvh=8Ty-iiarrzK)P-sn|');
</pre>

<p>WordPress Security Keys is a set of random variables that improve encryption of information stored in the user’s cookies. You should change them right after installation of your WordPress site. Use <a href="https://api.wordpress.org/secret-key/1.1/salt">this</a> site to get new set of fresh keys and then change them in you wp-config.php file.  </p>

<h1>Change file permissions</h1>

<p>Having secure file permissions may vary from your hosting provider, some may have secure permissions to your files, and some won't bother about this leaving you with default settings. Avoid having directories with 777 permissions, instead opt for 755 or 750. Also set files to 640 or 644 and wp-config.php to 600.</p>

<h1>Limit login attempts</h1>

<p>As I mentioned before, many attackers try to gain access with automated brute force attacks. Easiest way would be to use plugin that will forbid certain IP address from accessing the wp-admin page after certain amounts of failed logins. Plugin such as <a href="https://wordpress.org/plugins/all-in-one-wp-security-and-firewall/">All In One WP Security & Firewall</a> will allow you to change the URL of the login page - bots are programmed to look for wp-admin. </p>

<p>In addition to that use plugin limiting login attempts: </p>
<ul>
<li><a href="https://wordpress.org/plugins/limit-login-attempts-reloaded/">Limit Login Attempts Reloaded</a></li>
<li><a href="https://wordpress.org/plugins/login-lockdown/">Login Lockdown</a></li>
</ul>

<h1>Hide your WordPress version</h1>

<p>FIngerprinting WordPress is a huge issue, since from the attacker point of view - knowing the version will make it easier to find exploits. You can do this by adding this piece of code to your functions.php file. </p>

{% highlight php startinline %}
<?php
/* Hide WP version strings from scripts and styles
 * @return {string} $src
 * @filter script_loader_src
 * @filter style_loader_src
 */
function fjarrett_remove_wp_version_strings( $src ) {
     global $wp_version;
     parse_str(parse_url($src, PHP_URL_QUERY), $query);
     if ( !empty($query['ver']) && $query['ver'] === $wp_version ) {
          $src = remove_query_arg('ver', $src);
     }
     return $src;
}
add_filter( 'script_loader_src', 'fjarrett_remove_wp_version_strings' );
add_filter( 'style_loader_src', 'fjarrett_remove_wp_version_strings' );

/* Hide WP version strings from generator meta tag */
function wpmudev_remove_version() {
return '';
}
add_filter('the_generator', 'wpmudev_remove_version');
?>
{% endhighlight %}

<p>This will hide your WordPress version from all possible places. </p>

<h1>Last words</h1>

<p>Keeping your WordPress based site is very serious task, but following these tips will make it much safer than it was before, which undoubtedly will save you a lot of time in the future. But remember, there is no unhackable thing so always keep backups of your website, as in case of infection you will save more time than creating your site from scratch.  </p>

<p>I hope you learned something, from this article. As always remember and...</p>

<p>~ Stay safe!</p>
