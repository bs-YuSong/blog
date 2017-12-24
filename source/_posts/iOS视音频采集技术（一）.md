---
title: iOS视音频采集技术（一）
date: 2016-11-24 11:07:23
tags: 
	- 音频
	- 视频
	- iOS
categories: iOS
---

前言
===
这是一个关于视音频的系列文章，代码不追求质量，只把我认为最关键的部分提取出来，旨在方便学习，所以不建议直接用在项目之中。
本篇文章主要参考自：ObjC 中国

<!-- more -->

视频资源获取  
  UIImagePickerController使用  
  
  ```objc
  if UIImagePickerController.isSourceTypeAvailable(.camera) {
	  let imageVc = UIImagePickerController.init()
	  imageVc.sourceType = .camera
	  imageVc.mediaTypes = [kUTTypeMovie as String]
	  imageVc.videoQuality = .typeHigh
	  imageVc.delegate = self;
	  self.present(imageVc, animated: true, completion: nil)
  }
  ```
  
  AVFoundation视音频采集 
  AVCaptureDevice -> AVCaptureDeviceInput -> AVCaptureSession 
 
```objc
  AVCaptureSession *captureSession = [[AVCaptureSession alloc] init];
  self.captureSession = captureSession;
  // 摄像头设备配置
  AVCaptureDevice *cameraDevice;
  NSArray *devices = [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo];
  for (AVCaptureDevice *device in devices) {
  	if (device.positon == AVCaptureDevicePositionBack) {
  		cameraDevice = device;
  	}
  }
  AVCaptureDeviceInput *cameraDeviceInput = [[AVCaptureDeviceInput alloc] initWithDevice:cameraDevice error:nil];
  // 麦克风设备配置
  AVAudioSession *audioSession = [AVAudioSession shareInstance];
  [audioSession setCategory:AVAudioSessionCategoryPlayBack error:nil];
  NSArray *inputs = [audioSession availableInputs];
  AVAudioSessionPortDescription *builtInMic = nil;
  for (AVAudioSessionPortDescription *port in inputs) {
  	if ([port.portType isEqualToString:AVAudioSessionPortBuiltInMic]) {
  		builtInMic = port;
  		break;
  	}
  }
  for (AVAudioSessionDataSourceDescription *source in builtInMic.dataSources) {
    if ([source.orientation isEqualToString:AVAudioSessionOrientationFront]) {
        [builtInMic setPreferredDataSource:source error:nil];
        [audioSession setPreferredInput:builtInMic error:nil];
        break;
    }
  }
  AVCaptureDeviceInput *micDeviceInput = [[AVCaptureDeviceInput alloc] initWithDevice:[AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeAudio] error:nil];
  if ([captureSession canAddInput:cameraDeviceInput]) {
  	[captureSession addInput:cameraDeviceInput];
  }
  if ([captureSession canAddInput:micDeviceInput]) {
  	[captureSession addInput:micDeviceInput];
  }
  AVCaptureVideoPreviewLayer *previewLayer = [[AVCaptureVideoPreviewLayer alloc] initWithSession:captureSession];
  previewLayer.frame = self.view.bounds;
  [self.view.layer insertSublayer:previewLayer atIndex:0];
  self.captureSession.sessionPreset = AVCaptureSessionPresetHigh;
  [self.captureSession startRunning];
```
FileOutput视音频文件输出  
AVCaptureSession -> AVCaptureMovieFileOutput
	
```objc
AVCaptureMovieFileOutput *movieFileOutput = [AVCaptureMovieFileOutput new];
if ([self.captureSession canAddOutput:movieFileOutput]) {
	[self.captureSession addOutput:movieFileOutput];
}
// 开始录像
[self.movieFileOutput startRecordingToOutputFileURL:url recordingDelegate:self];
```

```objc
- (void)captureOutput:(AVCaptureFileOutput *)captureOutput didFinishRecordingToOutputFileAtURL:(NSURL *)outputFileURL fromConnections:(NSArray *)connections error:(NSError *)error {
	// 录像结束后调用
}
```
Writer视音频输出
captureSession -> captureVideo(or Audio)DataOutput
assetWriter -> assetWriterInput
captureSession 和 writer是平级，不需要关联
```objc
// Output
AVCaptureVideoDataOutput *videoDataOutput = [AVCaptureVideoDataOutput new];
videoDataOutput.alwaysDiscardsLateVideoFrames = NO;
[videoDataOutput setSampleBufferDelegate:self queue:dispatch_get_main_queue()];
videoDataOutput.videoSettings = nil;
[self.captureSession addOutput:videoDataOutput];
self.videoConnection = [videoDataOutput connectionWithMediaType:AVMediaTypeVideo];
self.videoSettings = [videoDataOutput recommendedVideoSettingsForAssetWriterWithOutputFileType:AVFileTypeQuickTimeMovie];

AVCaptureAudioDataOutput *audioDataOutput = [AVCaptureAudioDataOutput new];
[audioDataOutput setSampleBufferDelegate:self queue:dispatch_get_main_queue()];
[self.captureSession addOutput:audioDataOutput];
self.audioConnection = [audioDataOutput connectionWithMediaType:AVMediaTypeAudio];
self.audioSettings = [audioDataOutput recommendedAudioSettingsForAssetWriterWithOutputFileType:AVFileTypeQuickTimeMovie];
```
```objc
// Writer
AVAssetWriter *write = [[AVAssetWriter alloc] initWithURL:url fileType:AVFileTypeQuickTimeMovie error:nil];
    
AVAssetWriterInput *videoInput = [[AVAssetWriterInput alloc] initWithMediaType:AVMediaTypeVideo outputSettings:self.videoSettings];
videoInput.expectsMediaDataInRealTime = YES;
videoInput.transform = CGAffineTransformMakeRotation(M_PI_2);
if ([write canAddInput:videoInput]) {
    [write addInput:videoInput];
    self.videoInput = videoInput;
}

AVAssetWriterInput *audioInput = [[AVAssetWriterInput alloc] initWithMediaType:AVMediaTypeAudio outputSettings:self.audioSettings];
audioInput.expectsMediaDataInRealTime = YES;
if ([write canAddInput:audioInput]) {
    [write addInput:audioInput];
    self.audioInput = audioInput;
}
self.writer = write;
BOOL success = [self.writer startWriting];
if (!success) {
    NSError *error = self.writer.error;
    NSLog(@"error ---------- %@", error);
}
```
视频开始采集后需要在DataOutput的代理方法中使用writerInput采集buffer。
使用writer采集数据可定制性更高。