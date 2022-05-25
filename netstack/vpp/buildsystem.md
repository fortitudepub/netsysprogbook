# VPP build system

## pkg-deb-debug即打deb包的执行逻辑

在build-data/packages/vpp.mk中，有如下一个target：

```
vpp-package-deb: vpp-install
	@$(CMAKE) --build $(PACKAGE_BUILD_DIR)/vpp -- pkg-deb
	@find $(PACKAGE_BUILD_DIR) \
          -maxdepth 2 \
          \( -name '*.changes' -o -name '*.deb' -o -name '*.buildinfo' \) \
          -exec mv {} $(CURDIR) \;
```

该target会依赖于vpp-install，并且将执行cmake build的pkg-deb命令，该命令为一个无条件执行的CMAKE custom target，
将执行dpkg的命令对编译好的vpp进行deb打包操作。

上述custom target在src/pkg/CMakeLists.txt中定义。

### vpp-install

vpp-install target，由builkd-root/Makefile中的package_dependencies_fn生成，对于vpp这个package，该Makefile将
动态生成vpp-install依赖于vpp_install_depend这样的对象，并依赖于对应的vpp_configure/vpp_build目标。

而vpp_install/vpp_configure/vpp_build这些目录则在build-data/packages/vpp.mk中定义如下：

```
vpp_install = $(CMAKE) --build $(PACKAGE_BUILD_DIR) -- install | grep -v 'Set runtime path'
```

也即将再次调用cmake从而触发install命令执行，上述命令就切入cmake的构建逻辑了，也就是src/vpp/CMakeLists.txt所定义的命令。
