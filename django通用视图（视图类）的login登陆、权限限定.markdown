django的登陆限定和权限限定是view设计中很重要的一个组成。网上的教程大部分都是通过使用view函数的装饰器来实现，比如：

```python
@login_required(login_url='/')
@permission_required('can_access_assetView', login_url='/asset/error_403/')
def view_assetOperHistory(request):
    """
    # 查看修改历史
    """
    if request.method == 'GET':
        ...
```

相关的文章很多，用起来也很方便。

但是如果你有一个需求，需要使用django的视图类，即通用视图，来实现VIEW功能，则可能会有一点麻烦：网上教程大都是翻译或者指向django官方文档：https://docs.djangoproject.com/en/1.6/topics/class-based-views/intro/ 

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.views.generic import TemplateView

from .views import VoteView

urlpatterns = patterns('',
    (r'^about/', login_required(TemplateView.as_view(template_name="secret.html"))),
    (r'^vote/', permission_required('polls.can_vote')(VoteView.as_view())),
)
```

这种方法确实简单，直接在urls文件中对login和权限进行限定。但是有个问题，就是没法限定login的登陆后重定向位置，而且如果一个视图类，既需要login限定，又需要权限控制，那么都挤在一个地方，代码可读性很差，因此，最好是换一种实现方式，通过使用**多重继承**来实现login的限定。登陆限定类如下所示：


```python
class LoginRequiredMixin(object):
    """
    登陆限定，并指定登陆url
    """
    @classmethod
    def as_view(cls, **initkwargs):
        view = super(LoginRequiredMixin, cls).as_view(**initkwargs)
        return login_required(view, login_url='/')


class ViewAsset(LoginRequiredMixin, ListView):
    """
    具体的业务实现类
    """
    model = Group
    paginate_by = 10
    template_name = "asset_list.html"

```

非常简单，login限定完成，而且重定向的URL也有了。这时再在url文件中，对权限做限定，就一切OK了。

PS：django的通用视图封装的很棒，如果需要创建简单的应用，直接拿过来用很方便，但是如果是你的业务逻辑比较复杂，通用视图就不太适用了，还是老老实实用函数view的好。

```python
url(r'^assethistory/(?P<pk>\d+)/$', permission_required('asset.can_access_assetHistory', login_url='/asset/error_403/')(ViewAssetHistory.as_view()), name='viewAssetHistory'),
```


参考资料： https://stackoverflow.com/questions/10275164/django-generic-views-using-decorator-login-required

https://yq.aliyun.com/articles/45081

