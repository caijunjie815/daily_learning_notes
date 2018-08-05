# how to add markdown in Django
## objective
Implement markdown function (including code highlight) in both article and the comment in the Django-based blog website. Specifically, achieve the following functions:
  1. allow users to enter post content in markdown syntax in admin.
  2. allow users to comment and reply in markdown syntax.

## methods
There are two ways implementing this function, which are using **python modules** or **Javascript**.

### 1. use `markdown` module in Django
`markdown` package is able to render markdown-based content into html.
Here are three steps to apply this function using 'markdown' package:
#### 1.1 install packages
```
pip3 install markdown
pip3 install pygments
```
`pygments` is the module which allows you to generate css file for your codes, and so support code highlight.
#### 1.2 use `markdown` module
I have a `Post` model, which looks:
```python
# blog/models.py
class Post(models.Model):
	author = models.ForeignKey(User, on_delete=models.DO_NOTHING)
	title = models.CharField(max_length=100, unique=True)
	content = models.TextField(help_text='Enter your post content here')
	posted_time = models.DateField('date posted', auto_now=True)
	category = models.ForeignKey(Category, on_delete=models.SET_DEFAULT, default=1)
	tag = models.ManyToManyField(Tag)

	...
```
and I need markdown the `content` and `title` field if needed, so I add the following codes in `views.py`.
```python
#blog/views.py
...
import markdown

class PostView(DetailView):
  ... # ignore some codes

  def get_object(self, queryset=None):
    queryset = Post.objects.get(id=self.kwargs['pk'])
    # markdown the content of the post
    queryset.content = markdown.markdown(queryset.content, extensions=[
			'markdown.extensions.extra',
			'markdown.extensions.codehilite',
			'markdown.extensions.toc',
		])
    # also markdown the title if needed
    queryset.title = markdown.markdown(queryset.title, extensions=[
			'markdown.extensions.extra',
			'markdown.extensions.codehilite',
		])
    return queryset
```
also I need markdown the comment and reply part, add the following codes in `comment_sort` function:
```python
# blog/views.py
class PostView(DetailView):
  ... # ignore some codes

  def comment_sort(self, comments):
  ...
  for single_comment in self.comment_list:
    single_comment.content = markdown.markdown(single_comment.content, extensions=[
      'markdown.extensions.extra',
      'markdown.extensions.codehilite',
    ])
  return self.comment_list  # return sorted list of comments.
```
and the post lists in index page:
```python
#blog/views.py
class PostList(ListView):
	"""
	list view for index.html
	"""
	template_name = 'blog/index.html'
	# queryset = Post.objects.all()
	paginate_by = 5  # set numbers of posts per page.

	def get_queryset(self):
		results = Post.objects.all()
		for result in results:
			result.title = markdown.markdown(result.title, extensions=[
				'markdown.extensions.extra',
				'markdown.extensions.codehilite',
			])
			result.content = markdown.markdown(result.content, extensions=[
				'markdown.extensions.extra',
				'markdown.extensions.codehilite',
			])
		return results
```
#### 1.3 render template with code highlighting
add `safe` filter after the content what you want to render in templates, for example:
```html
<p class="markdown-content">{{ object.content|safe }}</p>
```
until now it is ok to render markdown-based content to html, however, the code is not highlighted yet, where the package `pygments` is deployed.
Choose a css style for codes from [my github](https://github.com/caijunjie815/django_blog/tree/master/blog/static/blog/css/highlights), and put it in the `blog/static/blog/css/highlights` directory, and then load it in `base.html`.
```html
<!-- blog/templates/blog/base.html -->
{% load static %}
<link rel="stylesheet" href="{% static 'blog/css/highlights/native.css' %}">
```
Finally, our codes is highlighted!
Note: remember add language name after the triple backticks.

### 2. use JavaScript
another way to implement markdown function is to use JavaScript, specifically, the jQuery library and a module called `marked`.
#### 2.1 add `marked` js into `base.html`
make sure `jQuery` have been in the `base.html`, like:
```html
<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
```
and then add the CDN of the `marked` or you can download it from its official website:
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/0.4.0/marked.min.js"></script>
```
#### 2.2 write a simple JavaScript to markdown
add a class `markdown-content` for the markdown content which will be rendered into html:
```html
<p class="markdown-content">{{ object.content|safe }}</p>
```
and then write the JS before the close tag of `</body>`:
```javascript
<script type="text/javascript">
    $(document).ready(function () {
        $('.markdown-content').each(function () {
            var content = $(this).text()
            console.log(content)
            var markdown_content = marked(content)
            console.log(markdown_content)
            $(this).html(markdown_content)
        })
    })
</script>
```
Explanation: get the text content with the class `markdown-content`, and use `marked` to transfer markdown-based text into html, and then replace the original content.
