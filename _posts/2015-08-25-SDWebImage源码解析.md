---
layout: post
title: SDWebImage源码解析10:41
category: iOS
comments: false
---

调用UIImageView+WebCache

```Objective-C
    - (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)**completedBlock**
{
  id <SDWebImageOperation> operation = [_SDWebImageManager.sharedManager_ downloadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) 
  {
    dispatch_main_sync_safe(^{
      wself.image = image;
      [wself setNeedsLayout];
      if (completedBlock && finished) {
        completedBlock(image, error, cacheType, url);
      }
    });
  }];
  >[self sd_setImageLoadOperation:operation forKey:@"UIImageViewImageLoad"];
}
```
>SDWebImageManager--单例

```Objective-C
    - (id <SDWebImageOperation>)downloadImageWithURL:(NSURL *)url
                                         options:(SDWebImageOptions)options
                                        progress:(SDWebImageDownloaderProgressBlock)progressBlock
                                       completed:(SDWebImageCompletionWithFinishedBlock)completedBlock 
{
    __block SDWebImageCombinedOperation *operation = [`SDWebImageCombinedOperation` new];
    __weak SDWebImageCombinedOperation *weakOperation = operation;

    @synchronized (self.runningOperations) {
        [self.runningOperations addObject:operation];
    }
    NSString *key = [self cacheKeyForURL:url];//根据url生成key

    operation.cacheOperation = [`self.imageCache` queryDiskCacheForKey:key done:^(UIImage *image, SDImageCacheType cacheType) 
    {
        if ((!image || options & SDWebImageRefreshCached) && (![self.delegate respondsToSelector:@selector(imageManager:shouldDownloadImageForURL:)] || [self.delegate imageManager:self shouldDownloadImageForURL:url])) 
        {
            `SDWebImageDownloaderOptions` downloaderOptions = 0;//根据配置赋值
            
            id <SDWebImageOperation> subOperation = [`self.imageDownloader` downloadImageWithURL:url options:downloaderOptions progress:progressBlock completed:^(UIImage *downloadedImage, NSData *data, NSError *error, BOOL finished) 
            {
                //一些失败的处理
                //获取图片成功的处理
                [self.imageCache storeImage:transformedImage recalculateFromImage:imageWasTransformed imageData:data forKey:key toDisk:cacheOnDisk];
                completedBlock(nil, nil, SDImageCacheTypeNone, YES, url);//主线程中执行

                [self.runningOperations removeObject:operation];//互斥执行
            }];
            operation.cancelBlock = ^{
                [subOperation cancel];
                [self.runningOperations removeObject:operation];//互斥执行
            };
        }
        else if (image) //在cache中找到image
        {
            completedBlock(nil, nil, SDImageCacheTypeNone, YES, url);//主线程中执行
            [self.runningOperations removeObject:operation];//互斥执行
        }
        else    //在cache中没找到image，且下载动作被delegate阻止
        {
            completedBlock(nil, nil, SDImageCacheTypeNone, YES, url);//主线程中执行
            [self.runningOperations removeObject:operation];//互斥执行
        }
    }];

    return operation;
}
```
