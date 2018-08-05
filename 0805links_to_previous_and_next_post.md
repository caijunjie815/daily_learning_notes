Assume we have a page displaying the details of an article, and we want to add links to last and next articles at the bottom of the article, just like this:

![](https://raw.githubusercontent.com/caijunjie815/daily_learning_notes/master/static/images/080501.png)

Firstly, we add the following codes in our `views.py`:
```python
class PostView(DetailView):
	...

	def get_context_data(self, **kwargs):
        ...

		# get previous and next article id
		current_id = self.kwargs['pk']
		context['previous_id'] = current_id - 1

		if current_id == Post.objects.latest('id').id:
			context['next_id'] = None
		else:
			context['next_id'] = current_id + 1
		return context
```
Here is what I do: pass `previous_id` and `next_id` into context of the object which will be further rendered in the template. Be Careful about two situations:
1. Current is the first post. In this case, the result of `current_id - 1` is the integer `0`. We do **NOT** need check whether `current_id` is the first one, because when we handle the `previous_id` in template later, the result `0` is what we expect. More explanation later.
2. Current is the last post. In this case, we need check whether `current_id` is the last one. If so, return `None`.

Next we add codes in templates:
```html
<div class='blog-post-link' style="margin-bottom: 1rem">
        <p>
            {% if previous_id %}
                <a class="btn blog-previous"
                   href="{% url 'blog:article' previous_id %}">Previous</a>
            {% else %}
                <a class="btn blog-previous">This is the first post</a>
            {% endif %}

            <a class="btn blog-overview" href="{% url 'blog:all-category' %}">Category List</a>

            {% if next_id %}
                <a class="btn blog-next" href="{% url 'blog:article' next_id %}">Next</a>
            {% else %}
                <a class="btn blog-next">This is the last post</a>
            {% endif %}
        </p>
</div>
```
Again for the first case (Current is the first post), as the `previous_id` is integer `0`, so the value of `if previous_id` would be `False`, therefore, the `else` statement is executed.

For the second case (Current is the last post), we get `None` as the value of `next_id`, which is `False` as the result of `if next_id`, thus a `else` statement is executed.

Here is the screenshoot for final view:
![](https://raw.githubusercontent.com/caijunjie815/daily_learning_notes/master/static/images/080502.png)
![](https://raw.githubusercontent.com/caijunjie815/daily_learning_notes/master/static/images/080503.png)
