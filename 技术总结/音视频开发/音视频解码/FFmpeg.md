---
创建时间: "2026-02-21 18:35:52"
作者: wangxiaoming
tags:
---
`FFmpeg` 整体是一个**模块化**的音视频处理框架，每个模块负责不同的功能，互相配合完成从“读文件”到“写文件”的完整流程。
#### 一、FFmpeg核心组成部分
##### 1）`libavformat` -- 封装/解封装

| 方法                             | 使用说明                             | 简单场景                         |
| ------------------------------ | -------------------------------- | ---------------------------- |
| `avformat_open_input()`        | 打开输入文件/流，探测并创建 `AVFormatContext` | 打开 `MP4/FLV/RTMP` 地址，准备读取数据  |
| `avformat_find_stream_info()`  | 解析文件头，获取流信息（视频/音频/字幕）            | 找出视频流索引，知道是 `H.264` 还是 `AAC` |
| `av_read_frame()`              | 从容器中读取一个压缩数据包 `AVPacket`         | 逐帧拿到 `H.264 NALU` 或 `AAC` 数据 |
| `avformat_write_header()`      | 写文件头，开始封装新文件                     | 把编码后的数据写成新的 `MP4/TS`         |
| `av_interleaved_write_frame()` | 写入一个 `AVPacket`，自动排序             | 把音视频包按 DTS 顺序写入文件            |
| `avformat_close_input()`       | 关闭文件，释放 `AVFormatContext`        | 结束读取，清理资源                    |

##### 2) `libavcodec` -- 编解码核心

| 方法                                | 使用说明                        | 简单场景                           |
| --------------------------------- | --------------------------- | ------------------------------ |
| `avcodec_find_decoder()`          | 根据 `codec_id`找到解码器          | 找到 `H.264` 解码器 `h264`          |
| `avcodec_find_encoder()`          | 根据 `codec_id`找到编码器          | 找到 `H.264` 编码器 `libx264`       |
| `avcodec_alloc_context3()`        | 创建编/解码器上下文 `AVCodecContext` | 为解码器分配配置空间                     |
| `avcodec_parameters_to_context()` | 把流的参数复制到 `AVCodecContext`   | 从 `AVStream->codecpar`复制宽高、码率等 |
| `avcodec_open2()`                 | 打开编/解码器，准备工作                | 启动解码器，可以开始 `send_packet`       |
| `avcodec_send_packet()`           | 把压缩包 `AVPacket`送入解码器        | 送 `H.264` NALU 进去              |
| `avcodec_receive_frame()`         | 从解码器取出解码后的 AVFrame          | 得到 YUV 帧                       |
| `avcodec_send_frame()`            | 把原始帧送入编码器（编码时）              | 送 YUV 帧进 H.264 编码器             |
| `avcodec_receive_packet()`        | 从编码器取压缩包 `AVPacket`         | 得到编码后的 H.264 NALU              |
| `avcodec_close()`                 | 关闭编/解码器，释放资源                | 结束编解码                          |
##### 3) `libavfilter` -- 滤镜

| 方法                               | 使用说明                    | 简单场景                                 |
| -------------------------------- | ----------------------- | ------------------------------------ |
| `avfilter_get_by_name()`         | 根据名字获取滤镜定义              | 获取 `"scale"`、`"overlay"`滤镜           |
| `avfilter_graph_alloc()`         | 创建滤镜图 `AVFilterGraph`   | 准备管理多个滤镜                             |
| `avfilter_graph_create_filter()` | 创建并初始化一个滤镜实例            | 创建一个 `scale`滤镜，设置输出宽高                |
| `avfilter_link()`                | 连接两个滤镜的输入/输出 pad        | 把 `buffer`滤镜 → `scale`→ `buffersink` |
| `avfilter_graph_config()`        | 检查并应用滤镜图的连接配置           | 启动滤镜链                                |
| `av_buffersrc_add_frame()`       | 往滤镜图输入一个 AVFrame        | 送解码后的 YUV 帧进入滤镜链                     |
| `av_buffersink_get_frame()`      | 从滤镜图输出端取出处理后的 `AVFrame` | 取到缩放/加水印后的帧                          |
##### 4) `libswscale`-- 视频缩放/格式转换

| 方法                  | 使用说明                      | 简单场景                          |
| ------------------- | ------------------------- | ----------------------------- |
| `sws_getContext()`  | 创建缩放/格式转换上下文 `SwsContext` | 创建 `YUV420P` → `RGB24` 的转换上下文 |
| `sws_scale()`       | 执行实际的缩放或像素格式转换            | 把 `1080P YUV` 缩放到 `720P RGB`  |
| `sws_freeContext()` | 释放 `SwsContext`           | 结束视频转换，回收内存                   |
##### 5) `libswresample` -- 音频重采样

| 方法                     | 使用说明                  | 简单场景                      |
| ---------------------- | --------------------- | ------------------------- |
| `swr_alloc()`          | 创建重采样上下文 `SwrContext` | 准备做 `44100Hz` → `48000Hz` |
| `swr_alloc_set_opts()` | 设置输入输出参数（采样率、声道、格式）   | 配置立体声 `S16` → 单声道 `FLT`   |
| `swr_init()`           | 初始化重采样器               | 启动音频转换                    |
| `swr_convert()`        | 执行重采样，转换音频数据          | 把 PCM 数据转为目标采样率           |
| `swr_free()`           | 释放 `SwrContext`       | 结束音频处理                    |
##### 6)`libavutil` -- 工具库

| 方法                                    | 使用说明                 | 简单场景                          |
| ------------------------------------- | -------------------- | ----------------------------- |
| `av_malloc()/ av_free()`              | 分配/释放 `FFmpeg` 管理的内存 | 替代 `malloc/free`，方便调试泄漏       |
| `av_log()`                            | 打印 `FFmpeg` 日志       | 输出调试信息，设置日志级别                 |
| `av_frame_alloc()/ av_frame_free()`   | 分配/释放 `AVFrame`      | 存放解码后的视频/音频帧                  |
| `av_packet_alloc()/ av_packet_free()` | 分配/释放 `AVPacket`     | 存放压缩数据包                       |
| `av_dict_set()`                       | 设置字典参数（如编码器选项）       | 设置 `"preset": "fast"`给 `x264` |
#### 二、场景：读取MP4
读 MP4 → 解码 → 缩放 → 编码 → 封装成新 MP4

> “在这个流程里，我先通过 `libavformat`​ 打开 `MP4` 并找到视频流，然后用 `libavcodec`​ 初始化解码器，循环读取压缩包并解码成 `YUV` 帧；接着用 `libswscale`​ 把帧缩放到目标分辨率；再用 `libavcodec`​ 编码成 `H.264`；最后用 `libavformat`​ 把编码后的数据封装成新的 `MP4`。资源管理全程用 `libavutil`​ 的内存函数和各个模块的 `close` 方法。”
##### 1）打开文件&找流（`libavformat`）
```c
avformat_open_input(&fmt_ctx, "input.mp4", NULL, NULL);
avformat_find_stream_info(fmt_ctx, NULL);
```
**解读**：
- **模块**：`libavformat`
- **作用**：解封装，打开 `MP4` 容器，读取文件头信息，找到里面的视频流和音频流。
- **关键点**：获取视频流索引 `video_stream_idx`，知道视频编码是 `H.264`、音频是 `AAC`。
##### 2) 初始化解码器 （`libavcodec`）
```c
codecpar = fmt_ctx->streams[video_stream_idx]->codecpar;
decoder = avcodec_find_decoder(codecpar->codec_id);
codec_ctx = avcodec_alloc_context3(decoder);
avcodec_parameters_to_context(codec_ctx, codecpar);
avcodec_open2(codec_ctx, decoder, NULL);
```
**解读**：
- **模块**：`libavcodec`
- **作用**：根据流的编码参数，找到对应的解码器（`H.264`），创建并配置解码器上下文。
- **关键点**：解码器准备好后，才能接收压缩包进行解码。
##### 3)读取压缩包&解码（`libavcodec`）
```c
av_read_frame(fmt_ctx, pkt);                 // libavformat
avcodec_send_packet(codec_ctx, pkt);         // libavcodec
avcodec_receive_frame(codec_ctx, frame);    // libavcodec
```
**解读**：
- **流程**：
    1. `av_read_frame`从容器里拿一个压缩包（`H.264 NALU`）。
    2. `avcodec_send_packet`把包送进解码器。
    3. `avcodec_receive_frame`取出解码后的原始帧（`YUV420P`）。
- **关键点**：解码得到的数据是未压缩的原始视频帧，用于后续处理。
##### 4) 视频缩放/格式转换（`libswscale`）
```c
sws_ctx = sws_getContext(src_w, src_h, AV_PIX_FMT_YUV420P,
                         dst_w, dst_h, AV_PIX_FMT_YUV420P,
                         SWS_BILINEAR, NULL, NULL, NULL);
sws_scale(sws_ctx, frame->data, frame->linesize, 0, src_h,
          dst_frame->data, dst_frame->linesize);
```
**解读**：
- **模块**：`libswscale`
- **作用**：把解码后的 YUV 帧缩放成目标分辨率（比如 1080P → 720P），或转换像素格式。
- **关键点**：很多编码器要求固定分辨率，所以这里做预处理。
##### 5) 初始化编码器（`libavcodec`）
```c
encoder = avcodec_find_encoder(AV_CODEC_ID_H264);
enc_ctx = avcodec_alloc_context3(encoder);
// 设置宽高、码率、帧率、GOP 等
avcodec_open2(enc_ctx, encoder, NULL);
```
**解读**：
- **模块**：`libavcodec`
- **作用**：创建 `H.264` 编码器，配置输出参数。
- **关键点**：编码参数决定最终视频质量和文件大小。
##### 6) 编码（`libavcodec`）
```c
avcodec_send_frame(enc_ctx, dst_frame);
avcodec_receive_packet(enc_ctx, enc_pkt);
```
**解读**：
- **流程**：
    1. 把缩放后的 `YUV` 帧送进编码器。
    2. 从编码器取出压缩包（`H.264 NALU`）。
- **关键点**：得到的新 `AVPacket`可以写入新文件。
##### 7) 封装成新文件（`libavformat`）
```c
avformat_alloc_output_context2(&out_fmt_ctx, NULL, NULL, "output.mp4");
avio_open(&out_fmt_ctx->pb, "output.mp4", AVIO_FLAG_WRITE);
avformat_new_stream(out_fmt_ctx, NULL);
// 写文件头
avformat_write_header(out_fmt_ctx, NULL);
// 写压缩包
av_interleaved_write_frame(out_fmt_ctx, enc_pkt);
// 写文件尾
av_write_trailer(out_fmt_ctx);
```
**解读**：
- **模块**：`libavformat`
- **作用**：创建新的 MP4 容器，把编码后的视频包按正确顺序写入，并封口。
- **关键点**：`av_interleaved_write_frame`会自动按 DTS 排序，保证播放正常。
##### 8) 资源释放（`libavutil + 各模块`）
```c
av_frame_free(&frame);
av_packet_free(&pkt);
avcodec_close(codec_ctx);
avcodec_close(enc_ctx);
avformat_close_input(&fmt_ctx);
avio_closep(&out_fmt_ctx->pb);
```
**解读**：
- **模块**：`libavutil`（内存管理）+ 其他模块的关闭函数
- **作用**：防止内存泄漏，释放所有分配的上下文和资源。