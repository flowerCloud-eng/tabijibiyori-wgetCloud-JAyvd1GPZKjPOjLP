
在智能设备普及和AI技术进步的推动下，用户对线上互动的质量、个性化以及沉浸式体验的追求日益增强。例如，对于热衷于图片编辑或视频制作的用户来说，他们需要一种快速而简便的方法来将特定主体从背景中分离出来。
HarmonyOS SDK [基础视觉服务](https://github.com "基础视觉服务")（Core Vision Kit）提供[主体分割能力](https://github.com "主体分割能力")，可以检测出图片中区别于背景的前景物体或区域（即"显著主体"），并将其从背景中分离出来，适用于需要识别和提取图像主要信息的场景，广泛使用于前景目标检测和前景主体分离的场景。


![image](https://img2024.cnblogs.com/blog/2396482/202501/2396482-20250110150407284-752695154.jpg)


### 适用场景


主体贴纸：从图片中提取显著性的主体，去掉背景。


背景替换：替换并提取出主体对象的背景。


显著性检测：快速定位图片中显著性区域。


辅助图片编辑：例如单独对主体进行美化处理。


### 开发步骤


1\.引用相关类添加至工程。



```
import { subjectSegmentation } from '@kit.CoreVisionKit';

```

2\.准备预处理的图片资源，将图片转换为PixelMap，并添加初始化和释放方法。



```
async aboutToAppear(): Promise<void> {
  const initResult = await subjectSegmentation.init();
  hilog.info(0x0000, 'subjectSegmentationSample', `Subject segmentation initialization result:${initResult}`);
}

async aboutToDisappear(): Promise<void> {
  await subjectSegmentation.release();
  hilog.info(0x0000, 'subjectSegmentationSample', 'Subject segmentation released successfully');
}

private async selectImage() {
  let uri = await this.openPhoto()
  if (uri === undefined) {
    hilog.error(0x0000, TAG, "uri is undefined");
  }
  this.loadImage(uri);
}

private openPhoto(): Promise<Array<string>> {
  return new Promise<Array<string>>((resolve, reject) => {
    let PhotoSelectOptions = new photoAccessHelper.PhotoSelectOptions();
    PhotoSelectOptions.MIMEType = photoAccessHelper.PhotoViewMIMETypes.IMAGE_TYPE;
    PhotoSelectOptions.maxSelectNumber = 1;
    let photoPicker: photoAccessHelper.PhotoViewPicker = new photoAccessHelper.PhotoViewPicker();
    hilog.info(0x0000, TAG, 'PhotoViewPicker.select successfully, PhotoSelectResult uri: ');
    photoPicker.select(PhotoSelectOptions).then((PhotoSelectResult) => {
      hilog.info(0x0000, TAG, `PhotoViewPicker.select successfully, PhotoSelectResult uri: ${PhotoSelectResult.photoUris}`);
      resolve(PhotoSelectResult.photoUris)
    }).catch((err: BusinessError) => {
      hilog.error(0x0000, TAG, `PhotoViewPicker.select failed with errCode: ${err.code}, errMessage: ${err.message}`);
      reject();
    });
  })
}

private loadImage(names: string[]) {
  setTimeout(async () => {
    let imageSource: image.ImageSource | undefined = undefined
    let fileSource = await fileIo.open(names[0], fileIo.OpenMode.READ_ONLY)
    imageSource = image.createImageSource(fileSource.fd)
    this.chooseImage = await imageSource.createPixelMap()
    hilog.info(0x0000, TAG, `this.chooseImage===${this.chooseImage}`);
  }, 100
  )
}

```

3\.实例化待分割的入参项VisionInfo，并传入待检测图片的PixelMap。



```
let visionInfo: subjectSegmentation.VisionInfo = {
  pixelMap: this.chooseImage,
};

```

4\.配置通用文本识别的配置项SegmentationConfig，包括最大分割主体个数、是否输出每个主体的分割信息，以及是否输出分割后的前景图。



```
let config: subjectSegmentation.SegmentationConfig = {
  maxCount: parseInt(this.maxNum),
  enableSubjectDetails: true,
  enableSubjectForegroundImage: true,
};

```

5\.调用imageSegmentation的ai.vision.doSegmentation接口，实现主体分割。



```
let data: subjectSegmentation.SegmentationResult = await subjectSegmentation.doSegmentation(visionInfo, config);
let outputString = `Subject count: ${data.subjectCount}\n`;
outputString += `Max subject count: ${config.maxCount}\n`;
outputString += `Enable subject details: ${config.enableSubjectDetails ? 'Yes' : 'No'}\n\n`;
let segBox : subjectSegmentation.Rectangle = data.fullSubject.subjectRectangle;
let segBoxString = `Full subject box:\nLeft: ${segBox.left}, Top: ${segBox.top}, Width: ${segBox.width}, Height: ${segBox.height}\n\n`;
outputString += segBoxString;

if (config.enableSubjectDetails) {
  outputString += 'Individual subject boxes:\n';
  if (data.subjectDetails) {
    for (let i = 0; i < data.subjectDetails.length; i++) {
      let detailSegBox: subjectSegmentation.Rectangle = data.subjectDetails[i].subjectRectangle;
      outputString += `Subject ${i + 1}:\nLeft: ${detailSegBox.left}, Top: ${detailSegBox.top}, Width: ${detailSegBox.width}, Height: ${detailSegBox.height}\n\n`;
    }
  }
}

```

**了解更多详情\>\>**


访问[基础视觉服务联盟官网](https://github.com "基础视觉服务联盟官网"):[milou加速器](https://xinminxuehui.org)


获取[主体分割开发指导文档](https://github.com "主体分割开发指导文档")


