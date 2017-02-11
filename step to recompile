# recompile-nginx-on-CentOS-to-support-HTTP-2
In CentOS command line, or an SSH terminal and enter the following command (note the uppercase letter V):
nginx -V
nginx version: nginx/1.10.3
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-file-aio --with-threads --with-ipv6 --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_ssl_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie',
So much content, in fact, we used to focus on is to look at compile nginx version of OpenSSL, is shown here:built with OpenSSL 1.0.1e-fips 11 Feb 2013
You can determine the version is openssl 1.0.1e-fips, clearly lower than 1.0.2 is not supported http2.

To clarify one point: although there are --with-http_v2_module this parameter, this means that in general see nginx supports http / 2, but in fact, this can only represent time VestaCP NginX installed on CentOS server, add the - with-http_v2_module this compilation arguments, but because openssl version 1.0.2 server itself is not enough, resulting in translation of the nginx can not support http / 2.

Then be clear: we do not need to upgrade the system comes with openssl, openssl upgrade just because a security risk. We only Download Source Package openssl 1.0.2, use it to recompile (equivalent to re-install) NginX can be on the server. If they become built with OpenSSL 1.0.2, it means being able to support http / 2 up.

Supplementary Update: After composing the first edition of this article, I was fortunate to have read the article @ Qu Guangyu "talk Nginx of the HTTP / 2 the POST the Bug" , so that must be used NginX v1.11 or later to avoid this bug. Therefore, we must first upgrade nginx version, and then recompile.

The following steps in CentOS 7 64bit + VestaCP 0.98-16 above test. I suggest you on the test server to operate again, and then perform the confirmation on the official server. In view of this upgrade requires a pause nginx service, so when you perform the official server, the best choice for late night or early morning visitors were fewer in order to reduce the impact on access to the front desk.

The first step: to upgrade an existing version 1.11 NginX

Because CentOS 7 default nginx's "stable version", but nginx v1.11 belong to the "main edition" instead of the stable, so we need to modify the source file centos7 of yum to use nginx main version.

Edit the nginx yum source file:

# vim /etc/yum.repos.d/nginx.repo
 

This line will baseurl amended as follows:

baseurl = https: //nginx.org/packages/mainline/centos/$releasever/$basearch/
 
Save the file, and then execute the following commands respectively:

# systemctl stop nginx
# yum clean all & yum upgrade nginx -y

Nginx will be automatically upgraded to version 1.11. Upon completion check the version obtained:

nginx -V
nginx version: nginx/1.11.9
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'


Step 2: Install dependencies

yum install gc gcc gcc-c++ pcre-devel zlib-devel make wget openssl-devel libxml2-devel libxslt-devel gd-devel perl-ExtUtils-Embed GeoIP-devel gperftools gperftools-devel libatomic_ops-devel perl-ExtUtils-Embed -y

Third step: switch to the source directory

Since we will compile the source code, so now into the source directory, the command is:
# cd /usr/local/src/

Part IV: download the source files

One by one the following order:

# wget http://www.openssl.org/source/openssl-1.0.2.tar.gz
# tar -xzvf openssl-1.0.2k.tar.gz

# NGINX_VERSION=1.11.
# wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
# tar -xvzf nginx-${NGINX_VERSION}.tar.gz

# wget https://hg.nginx.org/njs/archive/0.1.2.zip
# unzip 0.1.2.zip

Please note that the last set of commands 0.1.2  this string from the translation parameters previously saved in the "--add-dynamic-module = njs -0.1.2 / nginx" This is an extract. This string can not just make up their own, must be fully consistent with the original compile parameters. If your compiler is not this string parameters which control modifications.

Now enter nginx source directory:
# cd nginx-${NGINX_VERSION}/

Step Five: Modify the translation parameters

At the end of the first step in compiling copy the saved parameters, add the source path openssl 1.0.2 we have just unpacked:

--with-openssl=/usr/local/src/openssl-1.0.2

Note that this parameter with two hyphens (English characters) start, and must be between the front and parameters by at least one space.

 

Then, since we downloaded dynamic modules (dynamic module) source has a new path, to modify the original translation parameters in this line:

--add-dynamic-module = NJS-0.1.2 / nginx 

It read: 
--add-dynamic-module=/usr/local/src/njs-0.1.2/nginx
Now, we become a part of the translation parameters:

./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --with-openssl=/usr/local/src/openssl-1.0.2k --add-dynamic-module=/usr/local/src/njs-0.1.2/nginx

This part of the contents are saved to a text file for later use.

 

Step Six: Building Construction NginX

First, we need to stop nginx service, execute this command:

# systemctl stop nginx

Confirm currently in nginx 1.11.4 source directory, and then execute ./configure command, followed by the command ./configure compile all previously saved parameters, namely:

./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --with-openssl=/usr/local/src/openssl-1.0.2k --add-dynamic-module=/usr/local/src/njs-0.1.2/nginx

Finished will automatically return to the command line prompt, pay attention to see the results of the last few lines, if there is no error, execute the following two commands are:

# make
# make install 
if not success full you may see error like that:

/usr/bin/ld: /usr/local/src/openssl-1.0.2k/.openssl/lib/libssl.a(s23_meth.o): relocation R_X86_64_32 against `.rodata' can not be used when making a shared object; recompile with -fPIC
/usr/local/src/openssl-1.0.2k/.openssl/lib/libssl.a: could not read symbols: Bad value
collect2: ld returned 1 exit status
make[1]: *** [objs/nginx] Error 1
make[1]: Leaving directory `/root/nginx-http2/nginx-1.11.9'
make: *** [build] Error 2
to fixed error go to: 
vim /usr/local/src/nginx-1.11.9/objs/Makefile

find this line: && ./config --prefix=/usr/local/src/openssl-1.0.2k/.openssl no-shared \

and add -fPIC at the end: && ./config --prefix=/usr/local/src/openssl-1.0.2k/.openssl no-shared -fPIC \
save then run commend again.
# make
# make install
Wherein during the execution of commands make take a few minutes, you can get up and move around, stretch your body, or a cup of coffee. When make install is finished, it means nginx has been compiled successfully installed.

Step 7: Check results

Now re-check to see nginx version becomes built with openssl 1.0.2j:

# nginx -V
nginx version: nginx/1.11.9
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-11) (GCC) 
built with OpenSSL 1.0.2k  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' --with-openssl=/usr/local/src/openssl-1.0.2k --add-dynamic-module=/usr/local/src/njs-0.1.2/nginx

Open http / 2 option in the nginx configuration file:

Assuming your domain name is joomlagate.com, administrative user name of the site in vestacp is admin, then execute the following command to edit the nginx configuration file for this domain:

# vim /home/admin/conf/web/snginx.conf

In the beginning of the configuration file, the server {After that, you can see already configured SSL access:

listen      xx.xx.xx.xx:443 ssl;
Here xx.xx.xx.xx represent your domain name corresponding to the IP address, operating according to the actual situation.

Now only need to back ssl plus http2 option to become:

listen      xx.xx.xx.xx:443 ssl http2;
In this way, it opens up the nginx http2 function. However, do not forget the need to restart nginx to take effect. So also execute a command:

# systemctl restart nginx

After Now you can Google Chrome browser (version must be higher than v41.0) to access your site reception, press F12 (or click the Start menu "Developer Tools"), click on the network column, then refresh the page in protocol will be able to see a lot of H2 (shown below), on behalf of the page is transmitted through http2 protocol (If you only see http / 1.1 without h2, explain the failure, you can post a question to our forum, we work together to solve it).

