---
layout: post
title: SDWebImage源码解析
category: iOS
comments: false
---

#SDWebImage源码解析
调用
UIImageView+WebCache
{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
{% highlight iOS %}
  - (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock {
      id <SDWebImageOperation> operation = [SDWebImageManager.sharedManager downloadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
        dispatch_main_sync_safe(^{
                  wself.image = image;
                  [wself setNeedsLayout];
                if (completedBlock && finished) {
                    completedBlock(image, error, cacheType, url);
                }
            });
      }];
    [self sd_setImageLoadOperation:operation forKey:@"UIImageViewImageLoad"];
  }
{% endhighlight %}
