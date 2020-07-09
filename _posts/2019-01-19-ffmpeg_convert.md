---
title: "FFmpeg转码常用操作"
excerpt: "FFmpeg转码常用操作"
last_modified_at: 2019-01-19T12:55:00-14:30
header:
tags:
- FFmpeg
- 转码
toc: true
---

# FFmpeg转码常用操作
## 简介[^1]
[^1]: <https://zh.wikipedia.org/wiki/FFmpeg>

FFmpeg 是一个自由软件，可以运行音频和视频多种格式的录影、转换、流功能，包含了libavcodec——这是一个用于多个项目中音频和视频的解码器库，以及libavformat——一个音频与视频格式转换库。 “FFmpeg”这个单词中的“FF”指的是“Fast Forward”。

## 基本转换[^2]
[^2]: <https://opensource.com/article/17/6/ffmpeg-convert-media-file-formats>
FFmpeg的默认设置相当聪明，通常它会自动选择正确的编解码器和容器而无需任何复杂的配置。

例如，假设您有一个MP3文件并希望将其转换为OGG文件:

    ffmpeg -i input.mp3 output.ogg

此命令采用名为input.mp3的MP3文件，并将其转换为名为output.ogg的OGG文件。从FFmpeg的角度来看，这意味着将MP3音频流转换为Vorbis音频流并将此流包装到OGG容器中。您不必指定流或容器类型，因为FFmpeg为您找到了它。

这也适用于视频:

    ffmpeg -i input.mp4 output.webm

由于WebM是一种定义良好的格式，FFmpeg会自动知道它可以支持哪些视频和音频，并将流转换为有效的WebM文件。

根据您选择的容器，这并不总是有效。例如，像Matroska这样的容器可以处理几乎任何你想要放入的流，无论它们是否有效。这意味着命令:

    ffmpeg -i input.mp4 output.mkv

可能会导致文件具有与input.mp4相同的编解码器，这可能是您想要的，也可能不是。

## 选择你的编解码器
那么当你想使用像Matroska这样的容器(几乎可以处理任何流)但仍想修改输出的编码时，您可以使用-c标志选择所需的编解码器。
此标志允许您设置用于每个流的不同编解码器。 例如，要将音频流设置为Vorbis，您将使用以下命令:

    ffmpeg -i input.mp3 -c:a libvorbis output.ogg

更改视频和音频流也可以这样做:

    ffmpeg -i input.mp4 -c:v vp9 -c:a libvorbis output.mkv

这将使Matroska容器具有VP9视频流和Vorbis音频流，与我们之前制作的WebM基本相同。

命令`ffmpeg -codecs`将打印FFmpeg知道的每个编解码器。 此命令的输出将根据您安装的FFmpeg的版本而更改。

使用-target参数匹配行业标准，参数值可以是vcd、svcd、dvd、dv、dv50等，可能还需要加上电视制式作为前缀（pal-、ntsc-或film-）。如下:

    ffmpeg -i input.mp3 -target pal-dvd output.ogg

## 更改单个流

通常情况下，您的文件部分正确，只有一个错误格式的流。 重新编码正确的流可能非常耗时。FFmpeg可以帮助解决这种情况:

    ffmpeg -i input.webm -c:v copy -c:a flac output.mkv

此命令将视频流从input.webm复制到output.mkv ，并将Vorbis音频流编码为FLAC。

## 更换容器

前面的示例可以应用于音频和视频流，允许您从一种容器格式转换为另一种容器格式，而无需进行任何其他流编码:

    ffmpeg -i input.webm -c:av copy output.mkv

## 影响质量

现在我们已经掌握了编解码器，接下来的问题是:我们如何设置每个流的质量？

最简单的方法是改变比特率，这可能会或可能不会导致不同的质量。人类的视听能力并不像我们想的那样清晰明确。有时候改变比特率会对主观质量产生巨大影响。其他时候它可能只会改变文件大小。有时候如果不试一试就很难说出会发生什么。

要设置每个流的比特率，请使用-b标志，它以与-c标志类似的方式工作，除了您设置比特率的编解码器选项。

例如，要更改视频的比特率，您可以像这样使用它:

    ffmpeg -i input.webm -c:a copy -c:v vp9 -b:v 1M output.mkv

这将从input.webm复制音频(-c:a copy)，并将视频转换为比特率为1M/s(-b:v)的VP9编解码器(-c:v vp9)，所有这些都捆绑在一起一个Matroska容器(output.mkv)。

我们可以影响质量的另一种方法是使用-r选项调整视频的帧速率:

    ffmpeg -i input.webm -c:a copy -c:v vp9 -r 30 output.mkv

这创建了一个新的Matroska，其中复制了音频流，并且视频流的帧速率被强制为每秒30帧，而不是使用来自输入的帧速率(-r 30)。

您还可以使用FFmpeg调整视频的尺寸。 最简单的方法是使用预定的视频大小:

    ffmpeg -i input.mkv -c:a copy -s hd720 output.mkv

这会在输出中将视频修改为1280x720，但如果需要，您可以手动设置宽度和高度:

    ffmpeg -i input.mkv -c:a copy -s 1280x720 output.mkv

这将生成与上一条命令完全相同的输出。如果要在FFmpeg中设置自定义尺寸，请记住宽度参数(1280)在高度(720)之前。

调整帧速率和比特率是影响媒体质量的两种原始但有效的技术。 如果现有源的质量已经很低，则将这些值设置得非常高，无法提高现有源的质量。

更改这些设置对于快速减少高质量流以缩小文件大小非常有效。调整视频大小无法提高质量，但可以使其更适合平板电脑而非电视。将640x480视频的大小更改为4K不会改善它。

改变文件的质量是一个非常主观的问题，这意味着没有一种方法可以每次都有效。最好的方法是做一些小修改，并测试它是否看起来或听起来更合适。

## 修改流

    ffmpeg -i input.mkv -c:av copy -ss 00:01:00 -t 10 output.mkv

这将复制视频和音频流(-c:av copy)，并且修剪视频。 -t选项将剪切持续时间设置为10秒， -ss选项设置视频的开始点以进行剪裁，在本例中为一分钟(00:01:00)。 您可以更精确，而不仅仅是小时，分钟和秒，如果需要，可以缩短到几毫秒。


## 旋转视频[^3]
[^3]: <https://blog.csdn.net/coloriy/article/details/47447369>

想让视频顺时针旋转90度。这时候，可以使用-vf参数加入一个过滤器:

    ffmpeg -i input.mp3 -vf "rotate=90*PI/180" output.ogg
    
## 叠加图片

如果我们希望在一段视频上叠加一张图片。可以简单实现如下:

    ffmpeg -i input.mp3 -i image.png -filter_complex 'overlay' output.ogg


## 提取音频

有时你并不想要图像，你只需要音频。 幸运的是，在FFmpeg中使用-vn标志非常简单:

    ffmpeg -i input.mkv -vn audio_only.ogg

此命令仅从输入中提取音频，将其编码为Vorbis，并将其保存到audio_only.ogg中 。 现在你有一个独立的音频流。 您也可以使用-an和-sn标志以相同的方式去除音频和字幕流。

## 制作GIF

使用-an标志，类似于我们上面所做的，比创建动画GIF要好，如果你想制作一个没有音频的视频，但有很多地方支持不支持不同视频格式的GIF。

    ffmpeg -i input.mkv output.gif

此命令创建与输入文件具有相同尺寸的GIF。这通常不是一个好主意，因为相对于其他视频格式，GIF不能很好地压缩(根据我的经验，GIF将比源视频大8倍)。 使用-s选项将GIF的大小调整为稍微小一些可能会有所帮助，尤其是在输入源非常大的情况下，例如高清视频。

## 获取视频信息

有时您需要知道的是媒体文件内部信息，有几种工具可以做到这一点，最简单的是ffprobe工具:

    ffprobe -i input.mp4

比较强大的工具有[MediaInfo](https://mediaarea.net/en/MediaInfo)，运行命令`mediainfo inputFile.mkv`以人类可读的形式输出有关输入文件的信息列表。

## 更多

FFmpeg还有更多更强大的用法，具体请参阅[文档](http://ffmpeg.org/documentation.html)。

如果您使用的是带有图形界面的工具来转换多媒体，那么[Handbrake](https://handbrake.fr/)在Linux，Mac OS X和Windows上都是非常好用的。

