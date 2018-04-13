#WebRtc代码使用
1.环境配置
因为访问的是google的资源要配置代理 http 代理，和git代理

如：
export http_proxy=http://127.0.0.1:1087
export https_proxy=http://127.0.0.1:1087

git config --global http.proxy 'http://127.0.0.1:1080'

git config --global https.proxy 'http://127.0.0.1:1080'


2.安装 depot_tools,并添加到path
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

 export PATH=$PATH:/path/to/depot_tools
 
 注意：由于 depot_tools 依赖python2 请将环境默认python环境设为python2
 
 3.拉取代码
 
 mkdir webrtc-checkout
 cd webrtc-checkout
 fetch --nohooks webrtc
 gclient sync
 
 4.创建项目文件并编译
 查询可用参数
 gn args --list out/default
 
 target_os target_cpu
 
 gn gen out/default --args='is_debug=true target_os="mac" target_cpu="x64"' --ide=xcode

 编译：
 查询可以编译的target
 gn ls out/Default
 
 ninja -C out/Default all
 
 编译所有的模块
 
 
 增加自己的模块
 
 修改src/BUILD.gn
 在deps 中添加 
 ```
 "//tools/gn/tutorial:my_app"
 ```
 然后在目录 src/tool/gn/tutorial/中创建
 BUIDL.gn
 
 
 添加
 ```
 executable("my_app") {
  sources = [
    "hello_world.c",
  ]
  deps = [
      "//common_audio:common_audio",
  ]
}
```
和文件 hello_world.c


```#include "common_audio/vad/include/webrtc_vad.h"
#include <stdio.h>
#include <stdlib.h>
#define BUFFER_LEN 160

// 采样率为8000

void TestVAD(char *pAudioFile, char *pResFile, int rate, int nMode)
{
  // 创建
  VadInst* vad_ctx = NULL;
  if (NULL == (vad_ctx = WebRtcVad_Create())) {
    printf("WebRtcVad_Create error");
    return;
  }

  // 初始化
  if (WebRtcVad_Init(vad_ctx)) {
    printf("WebRtcVad_Init error");
    return;
  }

  // 设置模式

  if (WebRtcVad_set_mode(vad_ctx, nMode)) {
    printf("WebRtcVad_set_mode error");
    return;
  }

  FILE *fp = NULL;
  FILE *fpR = NULL;

  fp = fopen(pAudioFile, "rb");
  fpR = fopen(pResFile, "wb");

  fseek(fp, 0, SEEK_END);

  unsigned int len = ftell(fp);

  fseek(fp, 0, SEEK_SET);

  printf("file len is %d\n", len);
  short shBufferIn[BUFFER_LEN] = {0};

  while (1) {
    if (BUFFER_LEN != fread(shBufferIn, 1, BUFFER_LEN, fp)) {
      break;
    }
    if (WebRtcVad_ValidRateAndFrameLength(rate, BUFFER_LEN)) {
      printf("unValidRateAndFrameLength \n");
      return;
    }
    int nRet = WebRtcVad_Process(vad_ctx, 8000, shBufferIn, BUFFER_LEN);
    printf("\n%d\n", nRet);
    if (1 == nRet) {
      fwrite(shBufferIn, 1, BUFFER_LEN, fpR);
    }
  }

  fclose(fpR);
  fclose(fp);
  WebRtcVad_Free(vad_ctx);
}

int main() {
  printf("Hello, world.\n");
  printf("%lu", sizeof(short));
  TestVAD("/Users/liuhaiqiang/source/webrtc_checkout/output.raw", "/Users/liuhaiqiang/source/webrtc_checkout/output1.raw", 8000, 0);
  return 0;
}```


编译自己的模块

gn gen out/Default/
ninja -C out/Default my_app

编译完成后，可执行文件在 out/Default 下面

my_app 

./my_app 执行即可