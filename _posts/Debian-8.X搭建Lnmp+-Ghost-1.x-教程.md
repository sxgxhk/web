title: Debian 8.X搭建Lnmp+ Ghost 1.x 教程
categories: 网站建设
tags: [Ghost]
date: 2017-09-28 23:22:00
---
<p>其实类似的教程挺多的，尤其是Ghost 1.X版本之前的，真的不要太多，写这个教程也只是自己参照官网教程和烧饼博客之后，对自己的搭建经历作一个小小的回顾。因为本人还要折腾点别的小玩意，所以不仅要将Ghost博客搭建起来，还要将PHP环境等也一同搭建，便于日后的各种折腾。好了，废话不多说，开始啰嗦之旅吧。</p><ul><li>首先搭建Lnmp，可用的环境包有很多，比如军哥的Lnmp，还有最近比较火的OneinStack等等，个人比较喜欢OneinStack，觉得功能上比前者要丰富。具体的搭建步骤，请参考<a href="https://oneinstack.com/install/">官网</a>，一路配置下来估计也要半个小时左右，成功率100%（反正我折腾了这么多次，从来没有失败过），搭建好之后，建议先做个快照！<br />2017-8-19补充：还是把Oneinstack的搭建步骤照抄一下吧，给看的亲们省点时间233</li></ul><pre><code>yum -y install wget screen curl python #for CentOS/Redhat
# apt-get -y install wget screen curl python #for Debian/Ubuntu
wget http://aliyun-oss.linuxeye.com/oneinstack-full.tar.gz #阿里云经典网络下载
wget http://mirrors.linuxeye.com/oneinstack-full.tar.gz #包含源码，国内外均可下载
wget http://mirrors.linuxeye.com/oneinstack.tar.gz #不包含源码，建议仅国外主机下载
tar xzf oneinstack-full.tar.gz
cd oneinstack #如果需要修改目录(安装、数据存储、Nginx日志)，请修改options.conf文件
screen -S oneinstack #如果网路出现中断，可以执行命令`screen -R oneinstack`重新连接安装窗口
./install.sh #注：请勿sh install.sh或者bash install.sh这样执行</code></pre><p><img src="https://cdn.uu126.cn/201708/install_oneinstack.png" alt="install_oneinstack" title="install_oneinstack"></p><li><ul><li>开始搭建Ghost（过程可参照官网或烧饼博客）</li></ul></li><ol><li>更新系统并安装必要软件</li></ol><pre><code>apt-get update
apt-get upgrade
apt-get -t stretch-backports update
apt-get -t stretch-backports upgrade</code></pre><ol><li>安装 Node.js 6.x LTS<br />方法一：</li></ol><pre><code>curl -sL https://deb.nodesource.com/setup_6.x | bash -
apt-get install nodejs</code></pre><p>方法二：</p><pre><code>国外下载地址：
wget https://nodejs.org/download/release/v6.9.5/node-v6.9.5.tar.gz
国内下载地址：
wget https://npm.taobao.org/mirrors/node/v6.9.5/node-v6.9.5.tar.gz
tar zxvf node-v6.9.5.tar.gz
cd node-v6.9.5
./configure
make &amp;&amp; make install </code></pre><blockquote><p>2017-10-22补充，最新版的Ghost已经不支持Node V6.5版本了，所以嘛……得更新233</p></blockquote><p>推荐使用方法二，成功率高，如果使用方法一没有成功的话，还可以再使用方法二，成功与否，可以使用<br /><code>node -v </code>，如果能显示版本就说明Node.js搭建成功了</p><ol><li><p>配置Nginx（因为前面已搭建好Lnmp，所以这里直接配置站点文件即可）<br />先配置站点文件</p><pre><code>vi /usr/local/nginx/conf/vhost/ghost.conf</code></pre></li></ol><p>然后写入以下内容：</p><pre><code>server {
listen 80;
listen 443;
server_name 1984n.win;  #记得改成自己的域名
ssl on;
ssl_certificate /usr/local/nginx/conf/1984.crt; #SSL证书路径
ssl_certificate_key /usr/local/nginx/conf/1984.key;  #SSL证书路径
  location / {
      proxy_set_header   X-Real-IP $remote_addr;
      proxy_set_header   Host      $http_host;
      proxy_pass         http://127.0.0.1:2368;
  }
}</code></pre><p>保存好之后，记得重启一下Nginx。</p><ol><li>添加 ghost 运行用户（按照官方的说明，需要另建一个账户，名称随便，这里以<code>ghost</code>作为用户名来介绍。首先使用<code>adduser </code>而不是<code> useradd </code>命令添加用户，后者只能添加一个简单的无密码无执行权限的用户，前者是后者的加强版命令</li></ol><pre><code>adduser ghost</code></pre><p>然后系统会提示你输入两次密码，其他的一律回车即可。接着给予<code> ghost </code>用户<code> sudo </code>权限，这样这货就可以执行好多操作了。</p><pre><code>usermod -aG sudo ghost</code></pre><ol><li>安装 Ghost-CLI 和 Ghost 1.x<br />Ghost 从 1.0 开始，已经不需要其他第三方的软件来保持后台运行、更新、安装等操作，因为他们出了个命令行软件<code> Ghost-CLI </code>有了这货，我们再也不需要安装<code> pm2 </code>来保持后台运行，也不需要用<code> ghost-upgrade </code>来升级，因为他基本已经全部带了以前的功能。</li></ol><p>首先我们需要切换到<code> ghost </code>这个用户</p><pre><code>su - ghost</code></pre><p>输入密码回车,然后直接用 npm 安装<code> Ghost-CLI </code></p><pre><code>sudo npm i -g ghost-cli</code></pre><p>假设你的博客要放在<code> /var/www/ghost </code>目录，那么我们就创建一个并赋予权限</p><pre><code>sudo mkdir /var/www/ghost
sudo chown ghost:ghost /var/www/ghost</code></pre><p>进入这个目录就可以开始安装 Ghost 了</p><pre><code>cd /var/www/ghost
ghost install</code></pre><p>按照提示输入即可，如果要使用<code> sqlite3 </code>数据库的，可以将<code> ghost install </code>改为：</p><pre><code>ghost install local --db=sqlite3</code></pre><p>然后修改<code> /var/www/ghost/config.development.json </code>文件，修改自己的域名即可，保存好之后，再使用命令：<code> ghost stop &amp;&amp; ghost estart </code>命令强行关闭再启动（注意不是重启），看是否正确，正确的输出信息是类似这样的:</p><pre><code>ghost@sb-blog:/var/www/ghost$ ghost start
✔ Validating config
✔ Starting Ghost
You can access your blog at https://blog.1984n.win/
Ghost uses direct mail by default
To set up an alternative email method read our docs at https://docs.ghost.org/docs/mail-config</code></pre><p>没问题以后，就可以把这个文件改成正式环境的文件了:</p><pre><code>mv config.development.json config.production.json</code></pre><p>然后再使用命令：<code> ghost stop &amp;&amp; ghost estart </code>命令强行关闭再启动。</p><h4>注意:</h4><p>另外千万不要在 root 用户下执行 ghost 相关命令，否则 <code>/var/www/ghost/.ghostpid </code>这个文件的权限就变成<code> root </code>用户的了，导致后续会遇到<code> Message: 'EACCES: permission denied, open '/var/www/ghost/.ghostpid''</code></p><p>一切正常的话，就可以开始折腾Ghost了。</p><p>本文中搭建Ghost的部分步骤来自烧饼博客：<a href="https://sb.sb/debian-ubuntu-install-upgrade-ghost/">https://sb.sb/debian-ubuntu-install-upgrade-ghost/</a> ，还有Ghost官网教程：<a href="https://docs.ghost.org/docs/install">https://docs.ghost.org/docs/install</a> ，写完看电影咯。</p><p>2017-8-19补充：话说现在新版的Ghost编辑器还是蛮好用的，连小工具都有了，这在以前的0.7.4版本可是没有的，这也极大方便像我这种非代码控，点点即可上传图片、链接等等。</p><p><img src="https://cdn.uu126.cn/201708/ghost_blog02.jpg" alt="ghost_blog02" title="ghost_blog02"></p>