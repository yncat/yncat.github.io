---
layout: post

tags: tips
---

Google の [Resonance Audio](https://github.com/resonance-audio/resonance-audio?tab=readme-ov-file) を自前でビルドするときに、 git bash を使うように言われます。
が、それを導入するのはちょっと面倒くさいので、普通に msvc のコンソールからやるコマンドをメモっておきます。

```
mkdir build_api                                                                                             
cd build_api                                                                                                
cmake -DBUILD_RESONANCE_AUDIO_API:BOOL=ON ..                                                                
cmake --build . --config Release --target install                                                           
```

ゲームエンジンやらオーディオミドルウェアなんていうのはとりあえず知らないので C API だけのビルドです。私は C API だけあればてきとーにオーディオバックエンドに差し込むので問題ないのだ。
