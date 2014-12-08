svn_publisher
=============

一个基于SVN和rsync的线上代码发布系统

###这是一个旧项目
以前写过一个基于svn和rsync的代码发布系统，主要就是解决将代码发布到多台线上服务器的问题。  
需求本身很简单，一堆代码写好了，代码有改动了，如果依赖人工上传覆盖，或者打包到线上解压，都可能造成各种问题，万一运维疏忽，可能导致线上代码错误、版本不一致等等问题，如果线上服务器数量很多，这事儿就变得越来越危险，越来越不现实。  
并且，线上代码一旦发布之后，回滚、发布日志都是很麻烦的事情，那么就需要一个工具来管理。

于是我以前做的，就是拿一台能连上开发SVN服务器的机器，部署上SVN和rsync客户端，用php调用系统命令，执行一些诸如
    svn co
    svn update
    rsync 
的命令，在web端展示svn的版本历史，管理员选择一个版本之后，系统会把这个版本的代码发布到指定的服务器，并且记录完整的日志。

回滚也就是选择svn的某个历史版本，然后再次发布。

###重写的目的
重写主要还是为了练手，其实这个玩意做好了的话应该还蛮有用的，svn可以弥补rsync没有版本管理的弱点，用php和mysql做展示，可以让整个系统很傻瓜很好用，而且有完善的日志和回滚系统，让线上发布更安全。

这次打算把用户和发布项目的关联整理干净，提供完善的rsync配置说明，提供更完善的日志，以及一个看得过去的外观。

技术细节方面，打算更加熟悉一下git，以后这个项目可以考虑用git代替svn；用<del>YAF</del>Laravel框架代替yiiframework，使用php扩展代替php调用系统命令，让整个系统看起来不那么山寨，并且更健壮一些。  
好吧YAF看得我太纠结了，试试[Laravel](http://www.golaravel.com/)吧。。。


###优点
通过指定svn历史版本进行发布，好处是可以依赖svn的版本管理，又比svn钩子之类的更直观，对发布有更强的控制。  
通过分用户、分项目，使各项目之间各自独立，配置发布也比较简单。


###部署
这个系统是用于打通SVN和线上环境的，目前的设想是部署在2台服务器上，当然这2台服务器也可以合并在一起，主要取决于实际的网络情况。  

* __发布服务器：__ 发布服务器的作用是根据数据库中记录的任务，逐条执行，从SVN服务器获取项目代码，使用将代码推送到线上。发布服务器不必与实际使用者互通，而只要保证与SVN服务器、数据库以及线上各服务器能连通即可。  
软件方面，发布服务器需要安装rsync客户端，PHP 5.4 以上，需要部署本系统，需要安装SVN客户端，并且设置crontab `* * * * * php APP_ROOT/artisan task:run`。
* __Web端：__ Web端展示给实际使用者SVN的历史纪录，并将使用者设置的任务写入数据库，驱动发布脚本运行。所以Web端服务器需要能访问SVN服务器、数据库，还要提供Web界面给实际使用者。  
Web端需要PHP 5.4 以上，部署本系统，并需要安装SVN客户端。
* __DB：__ 数据库用于本系统管理各种日志记录、用户权限和各种项目信息的存储。发布服务器、Web端和DB可以放在同一台服务器上。

除此之外，要让发布服务器正常工作，必须配置好个项目的测试、正式服务器的rsync服务，以保证发布系统可以推送。
