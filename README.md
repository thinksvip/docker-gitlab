# docker-gitlab
## docker安装部署gitlab 使用非捆绑的Web服务器 

## 1. 项目目录
```
├── conf
│   └── nginx
│       ├── conf.d
│       │   ├── certs                #证书
│       │   └── www.site2.com.conf   #域名
│       └── nginx.conf
├── docker-compose.yml
├── env-example                      #默认配置文件
├── LICENSE
├── log
│   ├── gitlab
│   │   └── .gitignore
│   └── nginx
│       └── .gitignore
└── README.md
```
## 2. 快速使用
1. 本地安装`git`、`docker`和`docker-compose`。
2. `clone`项目：
```
    $ git clone git@github.com:thinksvip/docker-gitlab.git
```
3. 如果不是`root`用户，还需将当前用户加入`docker`用户组：
```
    $ sudo gpasswd -a ${USER} docker
```
4. 启动：
```
    $ cd docker-gitlab
    $ cp env-example .env
    $ docker-compose up
```
5. 访问在浏览器中访问：

 - 因为证书是自签证书，需要将 `www.site2.com` 添加入本机 `hosts`才可以访问。
 - [https://www.site2.com](https://site2.com)：自定义证书*https*站点，访问时浏览器会有安全提示，忽略提示访问即可

## 3. gitlab邮箱服务配置

- `vim .env ` 根据gitlab官方给出的[SMTP settings](https://docs.gitlab.com/omnibus/settings/smtp.html#aliyun-direct-mail)合理配置 `SMTP` 模块参数

## 4. 配置访问域名

```
    $ cd docker-gitlab
    $ sed -i 's/YOUR_DOMAIN_NAME=www.site2.com/YOUR_DOMAIN_NAME=your domain/g' .env
    $ cp /nginx/conf.d
    $ cp -a conf/nginx/conf.d/www.site2.com.conf conf/nginx/conf.d/your domain.conf
    $ sed -i 's/www.site2.com/your domain/g' conf/nginx/conf.d/your domain.conf
    $ mkdir conf/nginx/conf.d/certs/your domain
    $ cp your domain.key your domain.crt conf/nginx/conf.d/your domain.conf
    $ docker-composer up -d
```
## 5. 常见问题

- gitlab 500 报错 查看然后日志`/var/log/gitlab/gitlab-rails/production.log` 发现住住报错信息`OpenSSL::Cipher::CipherError ():`
```
$ cd docker-gitlab
$ docker-compose exec gitlab bash
$ less /var/log/gitlab/gitlab-rails/production.log

Started PATCH "/admin/application_settings/network" for 119.131.61.76 at 2019-08-27 03:28:14 +0000
Processing by Admin::ApplicationSettingsController#network as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"[FILTERED]", "application_setting"=>{"throttle_unauthenticated_enabled"=>"1", "throttle_unauthenticated_requests_per_period"=>"3600", "throttle_unauthenticated_period_in_seconds"=>"3600", "throttle_authenticated_api_enabled"=>"0", "throttle_authenticated_api_requests_per_period"=>"7200", "throttle_authenticated_api_period_in_seconds"=>"3600", "throttle_authenticated_web_enabled"=>"0", "throttle_authenticated_web_requests_per_period"=>"7200", "throttle_authenticated_web_period_in_seconds"=>"3600"}}
Completed 500 Internal Server Error in 32ms (ActiveRecord: 3.2ms)
  
OpenSSL::Cipher::CipherError ():
  
lib/gitlab/crypto_helper.rb:27:in `aes256_gcm_decrypt'
app/models/concerns/token_authenticatable_strategies/encrypted.rb:45:in `get_token'
app/models/concerns/token_authenticatable_strategies/base.rb:27:in `ensure_token'
app/models/concerns/token_authenticatable_strategies/encrypted.rb:32:in `ensure_token'
app/models/concerns/token_authenticatable.rb:38:in `block in add_authentication_token_field'
app/services/application_settings/update_service.rb:28:in `execute'
lib/gitlab/metrics/instrumentation.rb:161:in `block in execute'
lib/gitlab/metrics/method_call.rb:36:in `measure'
lib/gitlab/metrics/instrumentation.rb:161:in `execute'
app/controllers/admin/application_settings_controller.rb:124:in `perform_update'
app/controllers/admin/application_settings_controller.rb:17:in `block (2 levels) in <class:ApplicationSettingsController>'
lib/gitlab/session.rb:11:in `with_session'
app/controllers/application_controller.rb:450:in `set_session_storage'
lib/gitlab/i18n.rb:55:in `with_locale'
lib/gitlab/i18n.rb:61:in `with_user_locale'
app/controllers/application_controller.rb:444:in `set_locale'
lib/gitlab/middleware/rails_queue_duration.rb:27:in `call'
lib/gitlab/metrics/rack_middleware.rb:17:in `block in call'
lib/gitlab/metrics/transaction.rb:57:in `run'
lib/gitlab/metrics/rack_middleware.rb:17:in `call'
lib/gitlab/middleware/multipart.rb:103:in `call'
lib/gitlab/request_profiler/middleware.rb:17:in `call'
lib/gitlab/middleware/go.rb:20:in `call'
lib/gitlab/etag_caching/middleware.rb:13:in `call'
lib/gitlab/middleware/correlation_id.rb:16:in `block in call'
lib/gitlab/middleware/correlation_id.rb:15:in `call'
lib/gitlab/middleware/read_only/controller.rb:40:in `call'
lib/gitlab/middleware/read_only.rb:18:in `call'
lib/gitlab/middleware/basic_health_check.rb:25:in `call'
lib/gitlab/request_context.rb:26:in `call'
lib/gitlab/metrics/requests_rack_middleware.rb:29:in `call'
lib/gitlab/middleware/release_env.rb:12:in `call'
Started GET "/admin/application_settings/network" for 119.131.61.76 at 2019-08-27 03:28:17 +0000
Processing by Admin::ApplicationSettingsController#network as HTML
```
- gitlab 500 报错 解决方案

加密信息`/etc/gitlab/gitlab-secrets.json` 不存在，生成新的。

```
$ cd docker-gitlab
$ docker-compose exec gitlab bash
$ gitlab-rails console
$ ApplicationSetting.current.reset_runners_registration_token!
```
```
root@www:/# gitlab-rails console
--------------------------------------------------------------------------------
 GitLab:       12.2.1 (fa2d61ef521)
 GitLab Shell: 9.3.0
 PostgreSQL:   10.9
--------------------------------------------------------------------------------
Loading production environment (Rails 5.2.3)
irb(main):001:0> ApplicationSetting.current.reset_runners_registration_token!
=> true
irb(main):002:0> 

```

