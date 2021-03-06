#### 备忘
```txt
制作RPM包时不可以使用管理员的身份进行!

RPM生成要素
  1. 生成rpm所需要的文件列表或源代码   
  2. 根据文件列表或源码生成rpm规范 -> 即"spec"文件内描述
  3. 根据spec文件加工源码/文件的工具rpmbuild

xxx.src.rpm
  带有src后缀的rpm不是编译好的二进制程序
  其内部包含了源程序代码与SPEC文件!...
  需要使用rpmbuild命令将其编译为适合当前平台的rpm包之后再进行Install
```

#### 准备
```bash
[root@localhost ~]# yum -y install rpmdevtools pcre-devel       #包含rpmbuild，rpmdev-newspec，rpmdev-setuptree
[root@localhost ~]# cat ~/.rpmmacros                            #宏文件，此处指定RPM包制作的车间目录：'%_topdir'
%_topdir %(echo $HOME)/rpmbuild
[root@localhost ~]# rpmdev-setuptree                            #生成~/rpmbuild及子目录
[root@localhost ~]# tree rpmbuild                               #RPM包的制作车间（遵循一定的目录结构规范）
rpmbuild
├── BUILD                                         #解压后的文件所在（创建RPM时将自动在此目录执行某些操作）
├── RPMS                                          #存放制作完成后的二进制包（含以各平台命名的子目录及对应包）
├── SOURCES                                       #原材料位置，如源码包，文档...
├── SPECS                                         #存放管理rpm制作过程的描述文件（含宏及各阶段的定义和脚本等...）
└── SRPMS                                         #存放制作完成后的src格式rpm包（如：xxx.src.rpm，其没有平台依赖）

5 directories, 0 files
[root@localhost ~]# cd rpmbuild/SPECS/
[root@localhost SPECS]# rpmdev-newspec -o Name-version.spec     #生成默认的SPEC模板
Name-version.spec created; type minimal, rpm version >= 4.11.
[root@localhost SPECS]# cat Name-version.spec     # 示例 demo（k:v中的key可以%{key}的形式在spec中全局多引用..）
Name:           Name                              # 查询此处定义RPM的信息：rpm -qi xxx.rpm
Version:        x.x.x                             # 版本需严格匹配（%setup -q将自动使用这些宏进行一系列解压和cd操作）
Release:        1%{?dist}                         # "?": 即若存在dist宏（el5,el6,centos...）则替换
Summary:        A brief description of the package
Group:          Applications/Server
Distribution:   Linux                             # 发行版系列 
License:        GPLv2
Vender:         bluevitality <inmoonlight@163.com>
URL:            https://github.com/bluevitality
Source0:        %{name}.xxx                       # 明确说明源文件，默认将基于rpmbuild的SOURCES目录查找
Source1:        xxxxx                             # 默认将SOURCES下的源文件在rpmbuild的BUILD目录进行解压缩操作
Source2:        xxxxx

BuildRoot:      %{_topdir}/%{name}-%{version}-%{release}-root
# make install 时使用的虚拟根路径！（对OS不进行实际的安装操作）变量 $RPM_BUILD_ROOT 即对宏："%{BuildRoot}"的引用
#凡在此目录生成的文件必须做进rpm否则报错（可在install阶段先删除）
                                                  
BuildRequires:  gcc,automake,binutils,pcre-devel  # 制作时依赖的软件
Requires:       openssl,xxx,xxx                   # 安装时依赖的软件

Require(pre):   bash >= 3.0                       # 执行脚本时的依赖
Require(post):  python(flask) >= x.x              # 可指定依赖于特定语言的某个模块并指定依赖版本
Require(preun):
Require(postun):

%define   MACORS_NAME   VALUE                     # 用户自定义的SPEC宏，引用：%{MACORS_NAME}

%description                                      # rpm -qi xxx.rpm
Fill in the details about the package here
......

%prep                                             # 准备阶段
%setup -q                                         # 建议用"%setup -q"替代"%prep"的内容（此宏能够自动完成解压和cd）

%build                                            # 编译阶段
export DESTDIR=%{BuildRoot}
%configure                                        # rpmbuild --showrc | grep configure
make %{?_smp_mflags}

%install                                          # 安装阶段（在此阶段可实现删除不需要加入rpm包的文件）
rm -rf $RPM_BUILD_ROOT                            # 来自于宏定义，查看： rpmbuild --showrc | grep RPM_BUILD_ROOT
%make_install                                     # 相当于 %{__make} install DESTDIR=%{?buildroot}
%{__install} -p -d -m 0755 %{BuildRoot}/var/x     # 创建空目录
%{__install} -p -D -m 0755 %{Source1} %{BuildRoot}/etc/rc.d/init.d/daemon

%clean                                            # 安装完成后的清理阶段
rm -fr %{buildroot}                               

%files                                            # 文件阶段
%defattr(-, root, root, 0755)                     # 定义其下方对象的默认权限（-,user,group,perm）
%doc %{pecl_docdir}/%{pecl_name}                  # 指明文档文件，不指目标路径则位于/usr/share/doc/name-verion
%config(noreplace) %{_sysconfdir}/%{name}.conf    # 指明配置文件，此处设置位于%{BuildRoot}的：/etc/<name>.conf
/usr/local/bin/xxx                                # 将整个目录包含进rpm包中（此处的'/'即从从%{BuildRoot}开始的/）
%dir %attr(0755, redis, root) /lib/%{name}        # 引入空目录
%attr(0755,root,root) /etc/rc.d/init.d/daemon     # 引入启动脚本（%attr用来明确定义单个文件的属性）


%doc

%pre                                              # 在执行RPM的安装命令之前执行的shell脚本（脚本段可留空）

%post                                             # 安装之后执行

%preun                                            # 卸载之前执行

%postun                                           # 写在之后执行

%preun                                            # 脚本段，定义卸载前的动作，如杀掉进程.....

%changelog                                        # 变更日志（下面是摘来的例子）
* date +"%a %b %d %Y"  修改人  <邮箱>  本次版本-License修订号
- XXXXX ......

* Mon Aug 16 2010 Silas Sewell <inmoonlight@163.com> - 1.2.6-2
- Don't compress man pages
- Use patch to fix redis.conf

* Tue Jul 06 2010 Silas Sewell <inmoonlight@163.com> - 1.2.6-1
- Initial package
```
#### 构建
```txt
rpmbuild：
    -bl          检查spec中的%file段来查看文件是否齐全（检查有没有未被引入rpm包的文件）
    -ba          建立二进制格式rpm包&源码包
    -bb          建立二进制格式rpm包
    -bp          执行到 prep 阶段
    -bc          执行到 build 阶段
    -bi          执行到 install 阶段

制作：    
    cd /usr/src/redhat/SPECS/
    rpmbuild -ba nginx.spec
    将生成：/usr/src/redhat/RPMS/i386/nginx-1.2.1-1.el5.ngx.i386.rpm
    
src.rpm格式是rpm源码包，查看内容：    rpm2cpio filename.src.rpm | cpio -t
展开src.rpm格式文件内的SPEC文件：    rpm2cpio filename.src.rpm | cpio -id
使用src.rpm格式的文件制作成rpm包：    rpmbuild --rebuild filename.src.rpm  (默认将制作好的rpm包放至用户制作车间)
```

#### 软件包所属类别
```txt
Amusements/Games （娱乐/游戏）
Amusements/Graphics（娱乐/图形）
Applications/Archiving （应用/文档）
Applications/Communications（应用/通讯）
Applications/Databases （应用/数据库）
Applications/Editors （应用/编辑器）
Applications/Emulators （应用/仿真器）
Applications/Engineering （应用/工程）
Applications/File （应用/文件）
Applications/Internet （应用/因特网）
Applications/Multimedia（应用/多媒体）
Applications/Productivity （应用/产品）
Applications/Publishing（应用/印刷）
Applications/System（应用/系统）
Applications/Text （应用/文本）
Development/Debuggers （开发/调试器）
Development/Languages （开发/语言）
Development/Libraries （开发/函数库）
Development/System （开发/系统）
Development/Tools （开发/工具）
Documentation （文档）
System Environment/Base（系统环境/基础）
System Environment/Daemons （系统环境/守护）
System Environment/Kernel （系统环境/内核）
System Environment/Libraries （系统环境/函数库）
System Environment/Shells （系统环境/接口）
User Interface/Desktops（用户界面/桌面）
User Interface/X （用户界面/X窗口）
User Interface/X Hardware Support （用户界面/X硬件支持）
```
#### macros
```txt
%{_sysconfdir}        	/etc
%{_prefix}            	/usr
%{_exec_prefix}       	%{_prefix}
%{_bindir}            	%{_exec_prefix}/bin
%{_lib}               	lib (lib64 on 64bit systems)
%{_libdir}            	%{_exec_prefix}/%{_lib}
%{_libexecdir}        	%{_exec_prefix}/libexec
%{_sbindir}           	%{_exec_prefix}/sbin
%{_sharedstatedir}  	/var/lib
%{_datadir}           	%{_prefix}/share
%{_includedir}        	%{_prefix}/include
%{_oldincludedir}    	/usr/include
%{_infodir}           	/usr/share/info
%{_mandir}            	/usr/share/man
%{_localstatedir}    	/var
%{_initddir}          	%{_sysconfdir}/rc.d/init.d
%{_topdir}            	%{getenv:HOME}/rpmbuild
%{_builddir}         	%{_topdir}/BUILD
%{_rpmdir}            	%{_topdir}/RPMS
%{_sourcedir}         	%{_topdir}/SOURCES
%{_specdir}           	%{_topdir}/SPECS
%{_srcrpmdir}         	%{_topdir}/SRPMS
%{_buildrootdir}      	%{_topdir}/BUILDROOT
%{_var}               	/var
%{_tmppath}           	%{_var}/tmp
%{_usr}               	/usr
%{_usrsrc}            	%{_usr}/src
%{_docdir}            	%{_datadir}/doc
```
