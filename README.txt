
             Configuration of your autoproj build

- CMake
Since everything is CMake based, environment variables such as
CMAKE_PREFIX_PATH are always picked up. You can set them
in init.rb too, which will copy them to your env.sh script.

Because of cmake's aggressive caching behaviour, manual options
given to cmake will be overriden by autoproj later on. To make
such options permanent, add

  package('package_name').define "OPTION", "VALUE"

in overrides.rb. For instance, to set CMAKE_BUILD_TYPE for the rtt
package, do

  package('rtt').define "CMAKE_BUILD_TYPE", "Debug"

- Config files
There are various file that influence your build:

*.yml files: are simple 'key: value' pairs in the YAML format to set
             config options. This list is limited to what autoproj
	     knows.

*.rb  files: are ruby scripts that can influence any part of the 
             autoproj program, without modifying autoproj itself.
             This is only for advanced users that understand ruby
	     and the internals of autoproj.

- Configuration options

config.yml:  Save build configuration. You should not change it
             manually. If you need to change an option, run an
             autoproj operation with --reconfigure, as for
             instance
                  autoproj build --reconfigure

overrides.yml:
	     Override branch information for specific packages.
	     Most people leave this to the default, unless they
	     want to use a feature from an experimental branch.

- Influencing Autoproj ruby code:

init.rb:     Write in this file customization code that will get executed
	     before autoproj is loaded.

overrides.rb: 
	     Write in this file customization code that will get
	     executed after autoproj loaded.



编译步骤(Markdown 格式，可能显示有问题)：

  orocos的编译：

在 `~/.bashrc` 文件或者 `~/.profile` 文件中去掉之前orocos的环境变量，下载代码：

```
git clone https://github.com/orocos-toolchain/build.git
```

然后进入目录，执行：

```
./bootstrap.sh
选择：http, omniorb, xenomai
如果orogen编译有问题，可以尝试将 orogen、typelib、rtt_typelib、utilrb切换到toolchain-2.7（随着github上的更新，可能不需要这一步）：

cd orogen
git checkout toolchain-2.7
cd ../typelib
git checkout toolchain-2.7
cd ../rtt_typelib
git checkout toolchain-2.7
cd ../utilrb
git checkout toolchain-2.7
```

修改下列文件
 `rtt/config/FindBoost.cmake` 
 `utilmm/cmake/FindBoost.cmake`
 `orogen/lib/orogen/templates/config/FindBoost.cmake`
 `typelib/cmake/FindBoost.cmake`
 `/usr/share/cmake-2.8/Modules/FindBoost.cmake`

在每个文件最前面添加如下两行（否则会找不到高版本的boost）：
```
SET(Boost_ADDITIONAL_VERSIONS "1.60" "1.60.0")
SET(BOOST_ROOT "/usr/boost1_60_0")
```
或者尝试修改init.rb 中的环境变量(好像没有效果)


在命令行中执行：

```
export XENOMAI_ROOT_DIR=/usr/xenomai
```
找到文件 `rtt/rtt/transports/mqueue/binary_data_archive.hpp` :

```
编辑 rtt/rtt/transports/mqueue/binary_data_archive.hpp，删除其中一个函数的 BOOST_PFTO
```



然后开始编译：

```
. env.sh
autoproj fast-build --parallel=1
```

注意： 可能会出现内存不够编译失败的问题，设置交换分区以避免内存不足:
```
sudo fdisk -l 查看
sudo mount /dev/mmcblk1p1 /mnt
sudo swapon /mnt/swapfile
```

在编译 orogen 的时候可能失败，查看log文件：

```
可能出现问题如下：
1. 如果boost安装在特殊路径，运行时找不到boost动态库问题:  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/your/paht/to/boost
2. ruby 脚本问题 "cannot load such file -- Hoe":  gem install hoe
3. ruby 脚本问题 "cannot load such file -- Nokogiri":  gem install nokogiri
4. ruby 脚本问题 "cannot load such file -- bundler":  gem install bundler
5. 如果依赖项安装不上，可能是安装源的问题：
更新源：
    查看/etc/hosts, If the IP of ports.ubuntu.com is 91.189.88.140, change it to 91.189.88.151
    添加 127.0.0.1  localhost.localdomain  localhost
    执行下列命令
    sudo rm /var/lib/apt/lists/* -vf
    sudo apt-get update
```
