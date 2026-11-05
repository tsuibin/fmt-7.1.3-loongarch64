# fmt 7.1.3 on loongarch64

## 结论：版本必须锁定 7.1.3

系统 AOSC 自带 fmt 是 **v12**，与 Doris 2.1.11 的代码不兼容：
`Status::InternalError("...{}", msg)` 这类调用在 fmt 11+ 会报

    error: call to consteval ... is not a constant expression

因此必须编 Doris `thirdparty/vars.sh` 里指定的 **fmt 7.1.3**，不能用系统包。

## 构建

    cmake -B build -DCMAKE_INSTALL_PREFIX=$P -DCMAKE_BUILD_TYPE=Release \
          -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DFMT_TEST=OFF -DFMT_DOC=OFF -DFMT_INSTALL=ON .
    cmake --build build -j4 && cmake --install build

零补丁、零 loongarch 改动。产物：`installed/lib/libfmt.a`（约 1.2 MB）+ `include/fmt/`。
下载走 `https://gh-proxy.com/https://github.com/fmtlib/fmt/archive/refs/tags/7.1.3.tar.gz`。

## 验证

`grep FMT_VERSION installed/include/fmt/core.h` = 70103；已链接进 `doris_be`，
全量编译 0 错误、0 undefined symbol。

## 同期一并处理的头 / 依赖

`~/opt/build-fmt-deps.sh` 里还顺手做了两件事，都是 BE 编译期必需的：

    mkdir -p $P/include/lz4 && cp $P/include/lz4.h $P/include/lz4hc.h $P/include/lz4frame.h $P/include/lz4/
    cp concurrentqueue-1.0.3/{concurrentqueue.h,blockingconcurrentqueue.h} $P/include/

详见 `lz4-1.9.4-loongarch64` 与 `parallel-hashmap-1.3.8-loongarch64`。
