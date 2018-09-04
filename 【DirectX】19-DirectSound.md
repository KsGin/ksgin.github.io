---
title: 【DirectX】19-DirectSound
date: 2018-04-03 13:33:14
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[**Tutorial 14: Direct Sound**](http://www.rastertek.com/dx11tut14.html)

#### 学习记录

&emsp;&emsp;这篇文章中，我们将主要介绍 DX 中的音频输入 DirectSound 。与之前一样，这篇文章主要简单的介绍它的使用流程和完成一个播放的小例子。在此之前，如果对音频格式不太了解的话，可以看这里：[维基百科：音频格式](https://zh.wikipedia.org/wiki/%E9%9F%B3%E9%A2%91%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F)

<!--more-->

&emsp;&emsp;在使用 DirectSound 之前，需要包含 `dsound.h` 头文件以及链接 `dsound.lib` ，`dxguid.lib` ，`winmm.lib` 等库，如下：

```c++
#include <dsound.h>
#pragma comment(lib, "dsound.lib")
#pragma comment(lib, "dxguid.lib")
#pragma comment(lib, "winmm.lib")
```

&emsp;&emsp;我们这篇文章将会播放一个 .wav 格式的语音文件，大致分为以下步骤：

1. 初始化 DirectSound 设备以及主缓冲 DirectSoundBuffer
2. 读取 .wav 文件并校验
3. 使用文件信息配置辅助缓冲
4. 播放 DirectSoundBuffer

&emsp;&emsp;来试试看代码吧：

##### 初始化 DirectSound 设备以及主缓冲 DirectSoundBuffer

&emsp;&emsp;首先创建 DirectSound 指针变量且设置其属性：

```c++
IDirectSound8 * pSoundDevice = nullptr;
hr = DirectSoundCreate8(nullptr, &pSoundDevice, nullptr);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateSoundDevice", "ERROR", MB_OK);
	return hr;
}
hr = pSoundDevice->SetCooperativeLevel(hWnd, DSSCL_PRIORITY);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SetCooperativeLevel", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;这个和 DirectInput 还是比较像的。初始化好设备后，我们可以设置缓冲描述，并使用设备创建缓冲：

```c++
DSBUFFERDESC bd;
ZeroMemory(&bd, sizeof(bd));
bd.dwSize = sizeof(DSBUFFERDESC);
bd.dwFlags = DSBCAPS_PRIMARYBUFFER | DSBCAPS_CTRLVOLUME;
bd.guid3DAlgorithm = GUID_NULL;

IDirectSoundBuffer *pSoundPrimaryBuffer = nullptr;
hr = pSoundDevice->CreateSoundBuffer(&bd, &pSoundPrimaryBuffer, nullptr);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreatePrimaryBuffer", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;现在，我们可以配置这个缓冲的格式：

```c++
WAVEFORMATEX wf;
ZeroMemory(&wf, sizeof(wf));
wf.wFormatTag = WAVE_FORMAT_PCM;
wf.nSamplesPerSec = 44100;
wf.wBitsPerSample = 16;
wf.nChannels = 2;
wf.nBlockAlign = wf.wBitsPerSample / 8 * wf.nChannels;
wf.nAvgBytesPerSec = wf.nSamplesPerSec * wf.nBlockAlign;
wf.cbSize = 0;
hr = pSoundPrimaryBuffer->SetFormat(&wf);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SoundPrimaryBuffer::SetFormat", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;在这里我们创建了 WAV 文件的描述，并且将其设定为缓冲的格式。现在我们的基本缓冲对象就配置完了，用来存储音频信息的辅助缓冲会在读取和校验具体文件后才会进行。

##### 读取 .wav 文件并校验

&emsp;&emsp;在读取文件之前，我们需要创建一个音频的文件头结构体，以方便一会从 .wav 文件头中读取数据然后校验：

```c++
struct WaveHeader
{
	char chunkId[4];
	unsigned long chunkSize;
	char format[4];
	char subChunkId[4];
	unsigned long subChunkSize;
	unsigned short audioFormat;
	unsigned short numChannels;
	unsigned long sampleRate;
	unsigned long bytesPerSecond;
	unsigned short blockAlign;
	unsigned short bitsPerSample;
	char dataChunkId[4];
	unsigned long dataSize;
};
```

&emsp;&emsp;这一个结构体可能比较复杂，可以参考[维基百科：wav 文件格式](https://zh.wikipedia.org/wiki/WAV) ，有详细介绍 wav 的文件头各个参数，如下图（来自维基百科）：

![1](https://image.ibb.co/j759mc/Wave_format.png)

&emsp;&emsp;有了这个结构体，我们可以直接读取文件中的 Header 部分：

```c++
INT error;
FILE *filePtr = nullptr;
UINT count;
WaveHeader waveHeader;

error = fopen_s(&filePtr, "./sound.wav", "rb");
if (error) {
	MessageBox(nullptr, "ERROR::OpenWavFile", "ERROR", MB_OK);
	return false;
}
count = fread(&waveHeader, sizeof(waveHeader), 1, filePtr);
if (count != 1) {
	MessageBox(nullptr, "ERROR::ReadWavFileHeader", "ERROR", MB_OK);
	return false;
}
```

&emsp;&emsp;我们使用 fread 从内存层面直接将数据赋值到了 waveHeader 上，然后现在可以根据 waveHeader 的各个信息判断是否满足我们的要求：

```c++
if (waveHeader.chunkId[0] != 'R' || waveHeader.chunkId[1] != 'I' || waveHeader.chunkId[2] != 'F' || waveHeader.chunkId[3] != 'F') {
	MessageBox(nullptr, "ERROR::chunkId != RIFF", "ERROR", MB_OK);
	return false;
}
if (waveHeader.format[0] != 'W' || waveHeader.format[1] != 'A' || waveHeader.format[2] != 'V' || waveHeader.format[3] != 'E') {
	MessageBox(nullptr, "ERROR::format != WAVE", "ERROR", MB_OK);
	return false;
}
if (waveHeader.subChunkId[0] != 'f' || waveHeader.subChunkId[1] != 'm' || waveHeader.subChunkId[2] != 't' || waveHeader.subChunkId[3] != ' ') {
	MessageBox(nullptr, "ERROR::subChunkId != FMT", "ERROR", MB_OK);
	return false;
}
if (waveHeader.dataChunkId[0] != 'd' || waveHeader.dataChunkId[1] != 'a' || waveHeader.dataChunkId[2] != 't' || waveHeader.dataChunkId[3] != 'a') {
	MessageBox(nullptr, "ERROR::dataChunkId != data", "ERROR", MB_OK);
	return false;
}
if (waveHeader.audioFormat != WAVE_FORMAT_PCM) {
	MessageBox(nullptr, "ERROR::audioFormat != PCM", "ERROR", MB_OK);
	return false;
}
if (waveHeader.numChannels != 2) {
	MessageBox(nullptr, "ERROR::numChannels != 2", "ERROR", MB_OK);
	return false;
}
if (waveHeader.sampleRate != 44100) {
	MessageBox(nullptr, "ERROR::sampleRate != 44100", "ERROR", MB_OK);
	return false;
}
if (waveHeader.bitsPerSample != 16) {
	MessageBox(nullptr, "ERROR::bitsPerSample != 16", "ERROR", MB_OK);
	return false;
}
```

&emsp;&emsp;如果符合条件，我们就可以使用这个文件中的数据来创建一个存储信息的辅助缓冲区了。

##### 使用文件信息配置辅助缓冲

&emsp;&emsp;首先我们将数据读出来：

```c++
fseek(filePtr, sizeof(waveHeader), SEEK_SET);
UCHAR *waveData = new UCHAR[waveHeader.dataSize];
count = fread(waveData, 1, waveHeader.dataSize, filePtr);
if (count != waveHeader.dataSize) {
	MessageBox(nullptr, "ERROR::ReadSize != waveHeader.dataSize", "ERROR", MB_OK);
	return false;
}

error = fclose(filePtr);
if (error != 0) {
	MessageBox(nullptr, "ERROR::CloseFile", "ERROR", MB_OK);
	return false;
}
```

&emsp;&emsp;使用 fseek ，我们将文件指针定位到了 data 段的开始，然后将数据存入了 waveData 里。现在我们可以利用 waveData 创建一个临时缓冲：

```c++
WAVEFORMATEX waveFormatEx;
ZeroMemory(&waveFormatEx, sizeof(waveFormatEx));
waveFormatEx.wFormatTag = WAVE_FORMAT_PCM;
waveFormatEx.nSamplesPerSec = 44100;
waveFormatEx.wBitsPerSample = 16;
waveFormatEx.nChannels = 2;
waveFormatEx.nBlockAlign = waveFormatEx.wBitsPerSample / 8 * waveFormatEx.nChannels;
waveFormatEx.nAvgBytesPerSec = waveFormatEx.nSamplesPerSec * waveFormatEx.nBlockAlign;
waveFormatEx.cbSize = 0;

DSBUFFERDESC bufferDesc;
ZeroMemory(&bufferDesc, sizeof(bufferDesc));
bufferDesc.dwSize = sizeof(DSBUFFERDESC);
bufferDesc.dwFlags = DSBCAPS_CTRLVOLUME;
bufferDesc.lpwfxFormat = &waveFormatEx;
bufferDesc.dwBufferBytes = waveHeader.dataSize;
bufferDesc.guid3DAlgorithm = GUID_NULL;

IDirectSoundBuffer *pSoundTmpBuffer = nullptr;
hr = pSoundDevice->CreateSoundBuffer(&bufferDesc, &pSoundTmpBuffer, nullptr);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateSoundTmpBuffer", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;我们调用 `pSoundTmpBuffer` 的 `QueryInterface` 方法来创建辅助声音缓冲（在 dx 中，我们需要通过设备对象创建一个主缓冲。还需要至少一个用来存储声音信息的辅助缓冲，DirectSound 把辅助缓冲中的声音混合到主缓冲中）：

```c++
IDirectSoundBuffer8 *pSoundSecondaryBuffer = nullptr;
hr = pSoundTmpBuffer->QueryInterface(IID_IDirectSoundBuffer8, reinterpret_cast<LPVOID*>(&pSoundSecondaryBuffer));
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SoundTmpBuffer::QueryInterface", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;现在，我们有了我们的辅助缓冲对象，给它填充数据：

```c++
UCHAR *bufferPtr = nullptr;
ULONG bufferSize;
hr = pSoundSecondaryBuffer->Lock(0, waveHeader.dataSize, reinterpret_cast<LPVOID*>(&bufferPtr), &bufferSize, nullptr, 0, 0);	//Lock
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::pSoundSecondaryBuffer::Lock", "ERROR", MB_OK);
	return hr;
}

memcpy(bufferPtr, waveData, waveHeader.dataSize);	//copy

hr = pSoundSecondaryBuffer->Unlock(bufferPtr, bufferSize, nullptr, 0);	//UNLock
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::pSoundSecondaryBuffer::UNLock", "ERROR", MB_OK);
	return hr;
}
delete[] waveData;
waveData = 0;
```

&emsp;&emsp;这个时候已经可以进行播放了。

##### 播放 DirectSoundBuffer

```c++
hr = pSoundSecondaryBuffer->SetCurrentPosition(0);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::pSoundSecondaryBuffer::SetCurrentPosition", "ERROR", MB_OK);
	return hr;
}

hr = pSoundSecondaryBuffer->SetVolume(DSBVOLUME_MAX);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::pSoundSecondaryBuffer::SetVolume", "ERROR", MB_OK);
	return hr;
}

hr = pSoundSecondaryBuffer->Play(0, 0, 0);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::pSoundSecondaryBuffer::Play", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;源代码：[**DX11Tutorial-DirectSound**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-DirectSound)

