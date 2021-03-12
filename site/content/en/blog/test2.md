---
title: "Git 更換遠端伺服器倉庫網址URL"
date:   2020-08-21 00:00:01 -0800
categories:
  - blog
  - python
  - typing
authors: 
  - chenwei
---

## git switch remote URLs {#git-switch-remote-urls}

Git 更換遠端伺服器倉庫網址URL

1.  確認目前Git遠端伺服器網址： `git remote -v`

<!--listend-->

```text
git remote -v
origin  https://github.com/USERNAME/REPOSITORY.git (fetch)
origin  https://github.com/USERNAME/REPOSITORY.git (push)
```

1.  更換Git遠端伺服器位網址，使用： `git remote set-url`

<!--listend-->

```text
git remote set-url origin https://github.com/USERNAME/OTHERREPOSITORY.git
```

1.  再次確認Git遠端伺服器網址

<!--listend-->

```text
git remote -v
origin  https://github.com/USERNAME/OTHERREPOSITORY.git (fetch)
origin  https://github.com/USERNAME/OTHERREPOSITORY.git (push)
```

如果是使用SSH的存取網址，指令一樣是使用git remote set-url，再接上新的SSH URL就可以更換，指令如下：

```text
git remote set-url origin git@github.com:USERNAME/OTHERREPOSITORY.git
```

不管是要HTTP/HTTPS跟SSH，二種存取網址都是可以直接做更換，然後下次git push/ git fetch 就會到新設定的網址去了唷。

Resource:

-   <https://git-scm.com/book/zh-tw/v2/Git-%E5%9F%BA%E7%A4%8E-%E8%88%87%E9%81%A0%E7%AB%AF%E5%8D%94%E5%90%8C%E5%B7%A5%E4%BD%9C>


### Donec posuere augue in quam. {#donec-posuere-augue-in-quam-dot}

<https://blog.jiayu.co/2020/09/jolang/>

Hosting them yourself is also pretty easy. I recently moved away from Google Fonts for this site using this same process.

WriteFreely offers a growing number of tools to moderate your community of writers.

At Write.as, we're guided by a few core principles above all. They're not

Nullam eu ante vel est convallis dignissim.  Fusce suscipit, wisi nec facilisis facilisis, est dui fermentum leo, quis tempor ligula erat quis odio.  Nunc porta vulputate tellus.  Nunc rutrum turpis sed pede.  Sed bibendum.  Aliquam posuere.  Nunc aliquet, augue nec adipiscing interdum, lacus tellus malesuada massa, quis varius mi purus non odio.  Pellentesque condimentum, magna ut suscipit hendrerit, ipsum augue ornare nulla, non luctus diam neque sit amet urna.  Curabitur vulputate vestibulum lorem.  Fusce sagittis, libero non molestie mollis, magna orci ultrices dolor, at vulputate neque nulla lacinia eros.  Sed id ligula quis est convallis tempor.  Curabitur lacinia pulvinar nibh.  Nam a sapien.

Donec neque quam, dignissim in, mollis nec, sagittis eu, wisi.

```c
/* conversion functions */
static inline struct dwc2_hsotg_req *our_req(struct usb_request *req)
{
	return container_of(req, struct dwc2_hsotg_req, req);
}

static inline struct dwc2_hsotg_ep *our_ep(struct usb_ep *ep)
{
	return container_of(ep, struct dwc2_hsotg_ep, ep);
}

static inline struct dwc2_hsotg *to_hsotg(struct usb_gadget *gadget)
{
	return container_of(gadget, struct dwc2_hsotg, gadget);
}

static inline void __orr32(void __iomem *ptr, u32 val)
{
	dwc2_writel(dwc2_readl(ptr) | val, ptr);
}
```


#### Nullam rutrum. {#nullam-rutrum-dot}

Sed diam.

Nullam eu ante vel est convallis dignissim.  Fusce suscipit, wisi nec facilisis facilisis, est dui fermentum leo, quis tempor ligula erat quis odio.  Nunc porta vulputate tellus.  Nunc rutrum turpis sed pede.  Sed bibendum.  Aliquam posuere.  Nunc aliquet, augue nec adipiscing interdum, lacus tellus malesuada massa, quis varius mi purus non odio.  Pellentesque condimentum, magna ut suscipit hendrerit, ipsum augue ornare nulla, non luctus diam neque sit amet urna.  Curabitur vulputate vestibulum lorem.  Fusce sagittis, libero non molestie mollis, magna orci ultrices dolor, at vulputate neque nulla lacinia eros.  Sed id ligula quis est convallis tempor.  Curabitur lacinia pulvinar nibh.  Nam a sapien.
