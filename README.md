cloriConf
====

Suppose you programing in C++ and looking for a third-party configuration library, I suggest you have a try on cloriConf, which has character of both lightweight configuration library and distributed configuration center. 

## Features

The most significant features cloriConf exceeds general configuration library are 

* Configuration **format-independent**. CloriConf's access APIs are independent from configuration format. The currently supported formats are json and joml(James's Obvious Minimal Language, a superset of ini format), and more config format will be supported in future.
* Configuration **sourece-independent**. CloriConf not only can load various configuration format from file or string, but also accessing zookeeper is OK.

Multifunctional though cloriConf is, cloriConf's access API is designed to be as simple as a lightweight configuration library, see usage and API document for detail.

## Usage 

* Access INI-style/JOML-style configuration file  
common.ini: 
```C++
    [[adslot=xxx]]
      splash=123
      popUp=345
      [vta]
      splash=222 #comment test ; semicolon test
      popUp=3415
```
C++ code: 
```C++
    // SRC_LOCAL: load config from local file
    // FMT_JOML: parse as INI-style(JOML-style) config 
    // CMT_SHARP: take '#' as line comment identifier
    Config* conf = Config::instance()->Load("../conf/common.ini", SRC_LOCAL | FMT_JOML | CMT_SHARP);
    if (conf) {
        std::cout << "parse ini config file success" << std::endl;
    } else {
        std::cout << "parse ini config file failed"; 
        return;
    }
    std::string val1 = conf->GetString("adslot.vta.popUp");
    ConfNode* node = conf->GetConfNode("adslot.vta.splash");
    if (node) {
        std::cout << "adslot.vta.splash=" << node->AsString() << std::endl;
    }
    int val2 = conf->GetInt32("/adslot/vta/splash");
    std::cout << "val1=" << val1 << std::endl;
    std::cout << "val2=" << val2 << std::endl;
```
* Access JSON-style configuration file  
common.json:
```C++
    {
        "adslot":{
            "splash":"123",
            "popUp":"345",
            "vta":{
                "splash":"222",
                "popUp":"3415"
            }   
        }   
    }
```
C++ code: 
```C++
    // FMT_JSON: parse as JSON-style config
    std::string err_msg;
    Config* conf = Config::instance()->Load("../conf/common.json", SRC_LOCAL | FMT_JSON, &err_msg);
    if (conf) {
        std::cout << "parse json config file success" << std::endl;
    } else {
        std::cout << "parse json config file failed, " << err_msg << std::endl;
    }   
    std::string val1 = conf->GetString("adslot.vta.popUp");
    const ConfNode* node = conf->GetConfNode("adslot.vta.splash");
    if (node) {
        std::cout << "adslot.vta.splash=" << node->AsString() << std::endl;
        std::cout << "adslot.vta.popUp=" << conf->GetString("adslot.vta.popUp") << std::endl;
    }   

    // take 13456789 as default value when node "/adslot/vta/not_exist" not found
    int val2 = conf->GetInt32("/adslot/vta/not_exist", 13456789);
    std::cout << "val1=" << val1 << std::endl;
    std::cout << "val2=" << val2 << std::endl;
```
* Load config from zookeeper  
zk.ini(JOML-style):
```C++
    [zookeeper]
        # host=10.6.43.15:2181,10.6.43.16:2181,10.6.1.12:2181,10.6.1.13:2181
        host=localhost:2181
        timeout=3000
        interval=5000
        root=/online/commercial/ssp
    []
```
C++ code:
```C++
    // SRC_ZK: load config from zookeeper
    std::string err_msg;
    Config* conf = Config::instance()->Load("../conf/zk.ini", SRC_ZK, &err_msg);
    if (conf) {
        std::cout << "run test success" << std::endl;
    } else {
        std::cout << "run test failed, " << err_msg << std::endl;
        return;
    }
    std::string val1 = conf->GetString("rules/popUp");
    std::cout << "value of rules/popUp=" + val1 << std::endl;

    ConfNode* node = conf->GetConfNode("rules/splash");
    if (node) {
        for (ConfNode::ChildrenIterator iter = node->begin(); iter != node->end(); ++iter) {
            std::cout << "zookeeper config node, key=" << iter->name() << ", value=" << iter->AsString() << std::endl;
        }   
    }
```
## Installation

Before installation, you must be sure that
  * CloriConf currently passes the test in Linux operation system **only**. To support other OS, you may need to change CMakeLists.txt and some code.  
  * CloriConf is not a fully self-contained library, which has dependency on RapidJSON and zookeeper. To simplify installation of cloriConf, function for json/zookeeper support is disabled by default.  
To install cloriConf, you can run the following command in root path of cloriConf source code:
```C++
// (without support for JSON-style parser and zookeeper) 
mkdir build && cd build
cmake ..
make
sudo make install
```
To support JSON-style parser, you need to install [RapidJSON](https://github.com/Tencent/rapidjson) firstly, and then run
```C++
mkdir build && cd build
cmake .. -DENABLE_JSON=ON
make
sudo make install
```
To support zookeeper, you need to install [zookeeper](https://www.apache.org/dyn/closer.cgi) firstly, and then run
```C++
mkdir build && cd build
cmake .. -DENABLE_ZOOKEEPER=ON
make
sudo make install
```
Alternatively you can specify the install prefix by setting CMAKE_INSTALL_PREFIX, so a complete step seems like 
```C++
mkdir build && cd build
cmake .. -DENABLE_JSON=ON -DENABLE_ZOOKEEPER=ON -DCMAKE_INSTALL_PREFIX=/usr/local/third_party
make
sudo make install
```
After adding cloriConf to your program, you can compile like (assume cloriConf installed in /home/weijian/cloriconf)
```C++
g++ tutorial.cc -I/home/weijian/cloriconf/include -L/home/weijian/cloriconf/lib -lcloriconf -o main -std=c++11 -Wl,-rpath=/home/weijian/cloriconf/lib
```
## Build Up Zookeeper Dashboard by CloriConf
As cloriConf support loading config data from zookeeper, you can take it as a simple distributed configuration center to some extent. 
To simplify zookeeper management, cloriConf contain a zookeeper dashboard module in directory src/dashboard. To use cloriConf dashboard, a PHP runtime environment and nginx are required, and then you can add the following nginx config to nginx.conf and restart nginx
```PHP
    # set "/home/weijian/github/cloriConf" to your own directory
    location ~ \.php$ {
        root    /home/weijian/github/cloriConf/src/dashboard;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }   
    location ~ \.(css|js|html)$ {
        root        /home/weijian/github/cloriConf/src/dashboard;
    }
```
You may click [here](http://60.205.189.117/index.php?group_id=default&path=/online/commercial/ssp/rules) to see an example of cloriConf zookeeper dashboard.

## Who Is Using CloriConf? 

* [ofo 小黄车](http://www.ofo.so/#/) - ofo Inc., a Beijing-based bicycle sharing company

## Authors

* James Wei (weijianlhp@163.com)  
Please mail me in case of trouble with using cloriConf.

## API Reference 

### 1. Config* Config::instance()
```
    Get A config instance by singleton pattern.  
    Return value: a Config pointer to config instance.
```
### 2.Config* Load(const std::string& input, uint32_t mode, std::string* err_msg = NULL) noexcept
    Load config from string, local file or zookeeper(controlled by mode)

