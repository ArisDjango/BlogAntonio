## Chapter 1: Building a Blog Application 1
- Installing Django
- Creating an isolated Python environment
- Installing Django with pip
- Creating your first project
    - django-admin startproject mysite
    - cd mysite
    - python manage.py migrate
- Running the development server
    - python manage.py runserver
- Project settings
    - penjelasan core/settings.py
- Projects and applications
    - penjelasan struktur app
- Creating an application
    - python manage.py startapp blog
- Designing the blog data schema
    - model.py
- Activating the application
    - register di settings.py
- Creating and applying migrations
    - python manage.py makemigrations blog
    - python manage.py migrate
- Creating an administration site for models
- Creating a superuser
    - python manage.py createsuperuser
- The Django administration site
    - python manage.py runserver
    - http://127.0.0.1:8000/admin/
- Adding models to the administration site
    - admin.py
        ```
            from .models import Post

            admin.site.register(Post)
        ```
- Customizing the way that models are displayed

    - admin.py

    ```
        @admin.register(Post)
        class PostAdmin(admin.ModelAdmin):
            list_display = ('title', 'slug', 'author', 'publish', 'status')
            list_filter = ('status', 'created', 'publish', 'author')
            search_fields = ('title', 'body')
            prepopulated_fields = {'slug': ('title',)}
            raw_id_fields = ('author',)
            date_hierarchy = 'publish'
            ordering = ('status', 'publish')
    ```
- Working with QuerySets and managers
    - ORM
- Creating objects
    - Mencoba posting (melalui shell)
    - python manage.py shell
    ```
        >>> from django.contrib.auth.models import User
        >>> from blog.models import Post
        >>> user = User.objects.get(username='admin')
        >>> post = Post(title='Another post',
        ... slug='another-post',
        ... body='Post body.',
        ... author=user)
        >>> post.save()
    ```
- Updating objects
    ```
        >>> post.title = 'New title'
        >>> post.save()
    ```
- Retrieving objects (READ)
    ```
        >>> all_posts = Post.objects.all()
        >>> all_posts
    ```
- Using the filter() method
    ```
        Post.objects.filter(publish__year=2021)
        Post.objects.filter(publish__year=2021, author__username='admin')
    ```
    - atau
    ```
        Post.objects.filter(publish__year=2020) \
            .filter(author__username='admin')
    ```
- Using exclude() --> ascending/descending
    ```
        Post.objects.filter(publish__year=2020) \
        .exclude(title__startswith='Why')
    ```
- Using order_by()
    ```
        Post.objects.order_by('title')
        Post.objects.order_by('-title')
    ```
- Deleting objects
    ```
        post = Post.objects.get(id=1)
        post.delete()
    ```
- When QuerySets are evaluated
    ```
        QuerySets are only evaluated in the following cases:
        • The first time you iterate over them
        • When you slice them, for instance, Post.objects.all()[:3]
        • When you pickle or cache them
        • When you call repr() or len() on them
        • When you explicitly call list() on them
        • When you test them in a statement, such as bool(), or, and, or if
    ```
- Creating model managers
    - 'objects' adalah model manager default ketika membuat model
    - model manager bisa di custom
    ```
        class PublishedManager(models.Manager):
            def get_queryset(self):
                return super().get_queryset().filter(status='published')
    ```
    ```
        class Post(models.Model):
            # ...
            objects = models.Manager() # The default manager.
            published = PublishedManager() # Our custom manager.
    ```
- Building list and detail views
    - ORM
- Creating list and detail views ( Funtion Based View)
    ```
        from django.shortcuts import render, get_object_or_404

        def post_list(request):
            posts = Post.published.all()
            return render(request,'blog/post/list.html',{'posts': posts})

        def post_detail(request, year, month, day, post):
            post = get_object_or_404(Post, slug=post,
                                    status='published',
                                    publish__year=year,
                                    publish__month=month,
                                    publish__day=day)
            return render(request,'blog/post/detail.html',{'post': post})
    ```
- Adding URL patterns for your views
    - buat blog/urls.py
        ```
            from django.urls import path
            from . import views
            
            app_name = 'blog'

            urlpatterns = [
            
            path('', views.post_list, name='post_list'),
            path('<int:year>/<int:month>/<int:day>/<slug:post>/',
                views.post_detail,
                name='post_detail'),
            ]
        ```
- Canonical URLs for models
    ```
    def get_absolute_url(self):
        return reverse('blog:post_detail',
                       args=[self.publish.year,
                             self.publish.month,
                             self.publish.day, self.slug])
    ```
- Creating templates for your views
    - blog:
        ```
            templates/
                blog/
                    base.html
                    post/
                        list.html
                        detail.html
    ```
    - Static
        - buat blog/static/clog.css
- Adding pagination
    - views.py
    ```
        def post_list(request):
            object_list = Post.published.all()
            paginator = Paginator(object_list, 3) # 3 posts in each page
            page = request.GET.get('page')
            try:
                posts = paginator.page(page)
            except PageNotAnInteger:
                # If page is not an integer deliver the first page
                posts = paginator.page(1)
            except EmptyPage:
                # If page is out of range deliver last page of results
                posts = paginator.page(paginator.num_pages)
            return render(request,'blog/post/list.html',
                            {'page': page,'posts': posts})
    ```
    - buat blog/templates/pagination.html
        ```
            <div class="pagination">
                <span class="step-links">
                {% if page.has_previous %}
                    <a href="?page={{ page.previous_page_number }}">Previous</a>
                {% endif %}
                <span class="current">
                    Page {{ page.number }} of {{ page.paginator.num_pages }}.
                </span>
                {% if page.has_next %}
                    <a href="?page={{ page.next_page_number }}">Next</a>
                {% endif %}
                </span>
            </div>
        ```
    - buka blog/post/list.html, tempatkan tag pagination dibawah {content}  
    `{% include "pagination.html" with page=posts %}`
    - tes http://127.0.0.1:8000/blog/
- Using class-based views
    - selain menggunakan FBV bisa juga menggunakan class based view
    - tambahkan pada views.py
        ```
            class PostListView(ListView):
                queryset = Post.published.all()
                context_object_name = 'posts'
                paginate_by = 3
                template_name = 'blog/post/list.html'
        ```
    - bypass blog/urls.py
        ```
        path('', views.PostListView.as_view(), name='post_list'),
        ```
- Summary
## Chapter 2: Enhancing Your Blog with Advanced Features
- Sharing posts by email
    - Algoritma pengiriman email
        - buat form.py
        - buat views
        - by pass url
        - buat template untuk menampilkannya
- Creating forms with Django
    - Django punya 2 base class untuk form:
        - Form : standartd
        - ModelForm : forms dengan model instance
    - Buat blog/forms.py:
        ```
            from django import forms

            class EmailPostForm(forms.Form):
                name = forms.CharField(max_length=25)
                email = forms.EmailField()
                to = forms.EmailField()
                comments = forms.CharField(required=False, widget=forms.Textarea)
        ```
- Handling forms in views
    ```
        def post_share(request, post_id):
            # Mengambil post by id
            post = get_object_or_404(Post, id=post_id, status='published')
            if request.method == 'POST':
            # Form ter submit
                form = EmailPostForm(request.POST)
                if form.is_valid():
                # Form fields melalui validasi
                    cd = form.cleaned_data
                # ... mengirim email
            else:
                form = EmailPostForm()
            return render(request, 'blog/post/share.html', {'post': post,'form': form})
    ```
-  Sending emails with Django
    - konfigurasi Simple Mail Transfer Protocol (SMTP) server
        - settings.py
        - untuk mencoba di django shell kita butuh setting backend console (setelah dipastikan fungsi email berjalan, bisa di nonaktifkan)
        ```
        EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
        ```
        - SMTP server setting:
            ```
                EMAIL_HOST = 'smtp.gmail.com'
                EMAIL_HOST_USER = 'winandiaris@gmail.com'
                EMAIL_HOST_PASSWORD = 'xxxxxxxx'
                EMAIL_PORT = 587
                EMAIL_USE_TLS = True
            ```
        - Karena menggunakan gmail, Aktifkan 'Allow less secure apps'. ini akan mengijinkan teknologi dengan less secure bisa mengakses gmail.
        `https://myaccount.google.com/lesssecureapps`
        - selanjutnya Disable chaptcha
        ` https://accounts.google.com/displayunlockcaptcha`
        - cek koneksi di console, simulasikan pengiriman email
        - python manage.py shell
            ```
                >>> from django.core.mail import send_mail
                >>> send_mail('Django mail', 'This e-mail was sent with Django.', 'your_account@gmail.com', ['your_account@gmail.com'], fail_silently=False)
            ```
        - maka output =1 jika berhasil, selanjutnya:
        - Edit view.py, post_share(), menghandle email post
            - [view.py code](https://github.com/PacktPublishing/Django-3-by-Example/blob/master/Chapter02/mysite/blog/views.py)
        - tambahkan pada urls.py
        ```
            urlpatterns = [
                # ...
                path('<int:post_id>/share/',
                views.post_share, name='post_share'),
                ]
        ```
- Rendering forms in templates
    - buat file baru, templates/blog/post/share.html
        ```
            {% extends "blog/base.html" %}
            {% block title %}Share a post{% endblock %}

            {% block content %}
                {% if sent %}
                    <h1>E-mail successfully sent</h1>
                    <p>
                        "{{ post.title }}" was successfully sent to {{ form.cleaned_data.to }}.
                    </p>
                {% else %}
                    <h1>Share "{{ post.title }}" by e-mail</h1>
                    <form method="post">
                        {{ form.as_p }}
                        {% csrf_token %}
                        <input type="submit" value="Send e-mail">
                    </form>
                {% endif %}
            {% endblock %}
        ```
    - Edit the blog/post/detail.html
        - tempatkan code ini setelah `{{ post.body|linebreaks }}`
            ```
                <p>
                    <a href="{% url "blog:post_share" post.id %}">
                    Share this post
                    </a>
                </p>
            ```
    - cek 127.0.0.1:8000/blog/

- Creating a comment system
    1. membuat model untuk menyimpan comments
    2. membuat form untuk submit comments dan validasi input data
    3. membuat view untuk memproses form dan menyimpan comment ke database
    4. Edit post detail template untuk menampilkan daftar comments dan menampilkan form input comment
- Building a model
    - buka models.py
        ```
            class Comment(models.Model):
                post = models.ForeignKey(Post,
                                        on_delete=models.CASCADE,
                                        related_name='comments')
                name = models.CharField(max_length=80)
                email = models.EmailField()
                body = models.TextField()
                created = models.DateTimeField(auto_now_add=True)
                updated = models.DateTimeField(auto_now=True)
                active = models.BooleanField(default=True)

                class Meta:
                    ordering = ('created',)

                def __str__(self):
                    return f'Comment by {self.name} on {self.post}'
        ```
    - migrate
        - `python manage.py makemigrations blog`
        - `python manage.py migrate`
    - Menerapkan 'comment model' ke admin panel
    - admin.py
        ```
            from .models import Post, Comment

            @admin.register(Comment)
            class CommentAdmin(admin.ModelAdmin):
                list_display = ('name', 'email', 'post', 'created', 'active')
                list_filter = ('active', 'created', 'updated')
                search_fields = ('name', 'email', 'body')
        ```
    - cek 127.0.0.1:8000/admin
- Creating forms from models
    - forms.py
        ```
        from .models import Comment

        class CommentForm(forms.ModelForm):
            class Meta:
                model = Comment
                fields = ('name', 'email', 'body')
        ```
- Handling ModelForms in views
    - views.py pada post_detail()
        ```
        from .models import Post, Comment
        from .forms import EmailPostForm, CommentForm

        def post_detail(request, year, month, day, post):
        ...

            # List of active comments for this post
            comments = post.comments.filter(active=True)
            new_comment = None
            if request.method == 'POST':
                # A comment was posted
                comment_form = CommentForm(data=request.POST)
                if comment_form.is_valid():
                    # Create Comment object but don't save to database yet
                    new_comment = comment_form.save(commit=False)
                    # Assign the current post to the comment
                    new_comment.post = post
                    # Save the comment to the database
                    new_comment.save()
            else:
                comment_form = CommentForm()
            return render(request,
            'blog/post/detail.html',
                        {'post': post,
                        'comments': comments,
                        'new_comment': new_comment,
                        'comment_form': comment_form})
        ```
- Adding comments to the post detail template
    - Menampilkan comments pada `post/detail.html`
        • Menampilkan total comments sebuah post
        ```
        {% with comments.count as total_comments %}
            <h2>
                {{ total_comments }} comment{{ total_comments|pluralize }}
            </h2>
        {% endwith %}
        ```
        • Menampilkan daftar comments
        ```
        {% for comment in comments %}
            <div class="comment">
                <p class="info">
                    Comment {{ forloop.counter }} by {{ comment.name }}
                    {{ comment.created }}
                </p>
                {{ comment.body|linebreaks }}
            </div>
        {% empty %}
            <p>There are no comments yet.</p>
        {% endfor %}

        ```
        • Menampilkan form untuk users menginput comment baru
        ```
        {% if new_comment %}
            <h2>Your comment has been added.</h2>
        {% else %}
            <h2>Add a new comment</h2>
            <form method="post">
                {{ comment_form.as_p }}
                {% csrf_token %}
                <p><input type="submit" value="Add comment"></p>
            </form>
        {% endif %}
        
        ```
    - cek tampilan form comment http://127.0.0.1:8000/blog/
    
- Adding the tagging functionality
    - pip install django_taggit
    - register pada settings.py --> `taggit`
    - models.py:
        ```
        from taggit.managers import TaggableManager

            class Post(models.Model):
            # ...
            tags = TaggableManager()
        ```
    - python manage.py makemigrations blog
    - python manage.py migrate
    - coba fungsi tag melalui console
        - python manage.py shell
        ```
        >>> from blog.models import Post
        >>> post = Post.objects.get(id=1)
        >>> post.tags.add('music', 'jazz', 'django')
        >>> post.tags.all()
        <QuerySet [<Tag: jazz>, <Tag: django>, <Tag: music>]>
        ```
    - http://127.0.0.1:8000/admin/taggit/tag/
    - pada admin, Buka post/form edit, akan muncul form tag, yang bisa diinput lebih dari 1 dengan separasi coma
    - Menampilkan tag pada post
        - blog/post/list.html, tempatkan dibawah post title
        `<p class="tags">Tags: {{ post.tags.all|join:", " }}</p>`
        - Views.py
        ```
            from taggit.models import Tag

            def post_list(request, tag_slug=None):
                object_list = Post.published.all()
                tag = None

                if tag_slug:
                    tag = get_object_or_404(Tag, slug=tag_slug)
                    object_list = object_list.filter(tags__in=[tag])
                paginator = Paginator(object_list, 3) # 3 posts in each page
                # ...
        ```
        - render()
        ```
        return render(request, 'blog/post/list.html', {'page': page,
                                        'posts': posts,
                                        'tag': tag})
        ```
        
        - edit urls.py
            - aktifkan kembali path post_list yang menggunakan FBV
                - ` path('', views.post_list, name='post_list'),`
            - Tambahkan path tag
                ```
                path('tag/<slug:tag_slug>/', views.post_list name='post_list_by_tag'),
                ```
        - Edit `blog/post/list.html`
            - rubah pagination ke FBV `{% include "pagination.html" with page=posts %}`
            - untuk filter tag dihalaman utama, tempatkan diatas %for%
            ```
            <p class="tags">
            Tags:
            {% for tag in post.tags.all %}
                <a href="{% url "blog:post_list_by_tag" tag.slug %}">
                    {{ tag.name }}
                </a>
                {% if not forloop.last %}, {% endif %}
            {% endfor %}
            </p>
            ```

- Retrieving posts by similarity
    - Untuk mendapatkan saran post yang mirip ,berikut tahapannya:
        1. dapatkan semua tags untuk post
        2. dapatkan semua posts yang ter tag dengan salah satu tags
        3. Kecualikan postingan saat ini dari daftar itu untuk menghindari merekomendasikan postingan yang sama
        4. Urutkan hasil berdasarkan jumlah tag yang dibagikan dengan postingan saat ini
        5. Dalam kasus dua atau lebih posting dengan jumlah tag yang sama rekomendasikan posting terbaru
        6. Batasi kueri ke jumlah posting yang ingin Anda rekomendasikan
    - Implementasi:
    - pada views.py - post_detail()
        ```
        from django.db.models import Count

        # List of similar posts
        post_tags_ids = post.tags.values_list('id', flat=True)
        similar_posts = Post.published.filter(tags__in=post_tags_ids)\
                                    .exclude(id=post.id)
        similar_posts = similar_posts.annotate(same_tags=Count('tags'))\
                                    .order_by('-same_tags','-publish')[:4]

        ...
        # Pada render
        'similar_posts': similar_posts})
        ```

    - Pada blog/post/detail.html, tempatkan sebelum post comment
        ```
        <h2>Similar posts</h2>
        {% for post in similar_posts %}
            <p>
                <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
            </p>
        {% empty %}
            There are no similar posts yet.
        {% endfor %}
        ```
- Summary
## Chapter 3: Extending Your Blog Application
    • Membuat custom template tags dan filters
    • Membuat sitemap dan post feed
    • Implementasi full-text search dengan PostgreSQL

- Creating custom template tags and filters
    - referensi : https://docs.djangoproject.com/en/3.0/ref/templates/builtins/.

- Custom template tags
    - ada 2 macam:
        • simple_tag: memproses data dan mereturn string
        • inclusion_tag: memproses data dan mereturn render template


    - Buat directory blog/templatetags
    - buat file kosong __init__.py
    - buat file blog_tags.py
        ```
            from django import template
            from ..models import Post

            register = template.Library()

            @register.simple_tag
            def total_posts():
                return Post.published.count()
        ```
    - blog/templates/base.html
        ```
            {% load blog_tags %}
            ...
            <h2>My blog</h2>
                <p>This is my blog. I've written {% total_posts %} posts so far.</p>
        ```
        - Maka dibawah Blog title akan muncul jumlah total posts
    - Pada file blog_tags.py, tambahkan:
        ```
        @register.inclusion_tag('blog/post/latest_posts.html')
        def show_latest_posts(count=5):
            latest_posts = Post.published.order_by('-publish')[:count]
            return {'latest_posts': latest_posts}
        ```
    - Buat Template baru blog/post/latest_posts.html:
        ```
        <ul>
            {% for post in latest_posts %}
            <li>
                <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
            </li>
            {% endfor %}
        </ul>
        ```
    - Edit blog/base.html
        ```
        ...
        <p>This is my blog. I've written {% total_posts %} posts so far.</p>
                <h3>Latest posts</h3>
                {% show_latest_posts 3 %}
        ```
        - Maka akan muncul latest post pada side bar
    - Edit blog_tags.py
    ```
    from django.db.models import Count
    
    @register.simple_tag
    def get_most_commented_posts(count=5):
        return Post.published.annotate( total_comments=Count('comments')
        ).order_by('-total_comments')[:count]
    ```
    - Edit base.html
    ```
    <h3>Most commented posts</h3>
            {% get_most_commented_posts as most_commented_posts %}
            <ul>
                {% for post in most_commented_posts %}
                <li>
                    <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
                </li>
                {% endfor %}
            </ul>
    ```
    - Maka pada side bar akan muncul commented posts
    
- Custom template filters
    - contoh
    - {{ variable|my_filter }}
    - {{ variable|my_filter:"foo" }} --> dengan argument
    - {{ variable|filter1|filter2 }} --> multiple filter
    - doc : https://docs.djangoproject.com/en/3.0/ref/templates/builtins/#built-in-filterreference.
    - https://docs.djangoproject.com/en/3.0/howto/custom-templatetags/#writing-custom-template-filters.
    -
    - Install markdown --> `pip install markdown`
    - edit blog_tags.py
        ```
        from django.utils.safestring import mark_safe
        import markdown

        @register.filter(name='markdown')
        def markdown_format(text):
            return mark_safe(markdown.markdown(text))
        ```
    - Tambahkan Pada bagian atas setelah `extends` `blog/post/list.html` dan detail.html
    ` {% load blog_tags %} `
    - pada post/detail.html, replace {{ post.body|linebreaks }} dengan {{ post.body|markdown }}
    - pada post/list.html, replace {{ post.body|truncatewords:30|linebreaks }} dengan {{ post.body|markdown|truncatewords_html:30 }}
    - Coba Buat postingan menggunakan format markdown
        - Buka http://127.0.0.1:8000/admin/blog/post/add/
        - maka jika dilihat format akan menjadi markdown
        - doc: https://daringfireball.net/projects/markdown/basics.

- Adding a sitemap to your site
    - doc : https://docs.djangoproject.com/en/3.0/ref/contrib/sitemaps/
    - settings.py
        ```
        SITE_ID = 1

        # Application definition
        INSTALLED_APPS = [
        # ...
        'django.contrib.sites',
        'django.contrib.sitemaps',
        ]
        ```
        python manage.py migrate

    - Buat baru blog/sitemaps.py
        ```
        from django.contrib.sitemaps import Sitemap
        from .models import Post

        class PostSitemap(Sitemap):
            changefreq = 'weekly'
            priority = 0.9
            
            def items(self):
                return Post.published.all()
            def lastmod(self, obj):
                return obj.updated
        ```
    - BUka core/urls.py
        ```
        from django.urls import path, include
        from django.contrib import admin
        from django.contrib.sitemaps.views import sitemap
        from blog.sitemaps import PostSitemap

        sitemaps = {
        'posts': PostSitemap,
        }

        urlpatterns = [
            path('admin/', admin.site.urls),
            path('blog/', include('blog.urls', namespace='blog')),
            path('sitemap.xml', sitemap, {'sitemaps': sitemaps},
            name='django.contrib.sitemaps.views.sitemap')
            ]
        ```
    - BUka http://127.0.0.1:8000/sitemap.xml, akan muncul bentuk xml dari site kita, xml berpengaruh pada efektivitas search engine

    - http://127.0.0.1:8000/admin/sites/site/ untuk menambah daftar domain yang digunakan di sitemap


- Creating feeds for your blog posts
    - ref:  Django syndication feed framework at https://docs.djangoproject.com/en/3.0/ref/contrib/syndication/
    - secara dinamis akan mengenerate RSS atau atom feed. web feed data format(biasanya XML) memungkinkan user mendapatkan update feed menggunakan feed agragator.
    - buat file baru, blog/feeds.py
        ```
        from django.contrib.syndication.views import Feed
        from django.template.defaultfilters import truncatewords
        from django.urls import reverse_lazy
        from .models import Post
        class LatestPostsFeed(Feed):
        title = 'My blog'
        link = reverse_lazy('blog:post_list')
        description = 'New posts of my blog.'
        def items(self):
        return Post.published.all()[:5]
        def item_title(self, item):
        return item.title
        def item_description(self, item):
        return truncatewords(item.body, 30)
        ```
    - tambahkan pada blog/urls.py
        ```
        from .feeds import LatestPostsFeed

        urlpatterns = [
        # ...
        path('feed/', LatestPostsFeed(), name='post_feed'),
        ]
        ```
    - buka http://127.0.0.1:8000/blog/feed/, maka akan melihat format xml
    - buka blog/base.html, tempatkan pada side bar, dibawah total post
        ```
        <p>
        <a href="{% url "blog:post_feed" %}">Subscribe to my RSS feed</a>
        </p>
        ```
    - Buka http://127.0.0.1:8000/blog/

- Adding full-text search to your blog
    - pencarian dalam blog bisa menggunakan cara filter ORM, misal
        ```
        from blog.models import Post
        Post.objects.filter(body__contains='framework')
        ```
    - Untuk pencarian secara kompleks, bisa menggunakan fitur dari postgreSQL dan modul `django.contrib.postgres`
    - docs: https://www.postgresql.org/docs/12/static/textsearch.html

- Installing PostgreSQL
    - https://www.postgresql.org/download/
    - pip install psycopg2-binary
    - Buat user & password baru, dan memberi hak akses 
        ```
        psql -U postgres
        CREATE USER aris;
        ALTER USER aris PASSWORD 'aris1985';
        ALTER USER aris CREATEDB;
        ```
    - buat database melalui user 'aris'
        ```
        createdb -U aris blog;
        ```
    - settings.py
    ```
    DATABASES = {
        'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'blog',
        'USER': 'blog',
        'PASSWORD': '*****',
        }
    ```
    - python manage.py migrate
    - python manage.py createsuperuser
    - http://127.0.0.1:8000/admin/
    - karena ganti database postgres, semua post lama hilang. sekarang buat post baru agar search fitur bisa di terapkan
- Simple search lookups
    - Sekarang bisa melakukan search single field, contoh:
        ```
        from blog.models import Post
        Post.objects.filter(body__search='django')
        ```
- Searching against multiple fields
    - search multiple fields, memungkinkan  search terhadap title dan body dari Post model
    - contoh :
        ```
        from django.contrib.postgres.search import SearchVector
        from blog.models import Post

        Post.objects.annotate( search=SearchVector('title', 'body'),
            ).filter(search='django')
        ```

- Building a search view
    - Membuat view untuk search post
    - edit forms.py
        ```
        class SearchForm(forms.Form):
            query = forms.CharField()
        ```
    - edit views.py
        ```
        from django.contrib.postgres.search import SearchVector
        from .forms import EmailPostForm, CommentForm, SearchForm

        def post_search(request):
        form = SearchForm()
        query = None
        results = []
        if 'query' in request.GET:
            form = SearchForm(request.GET)
            if form.is_valid():
                query = form.cleaned_data['query']
                results = Post.published.annotate(
                    search=SearchVector('title', 'body'),
                    ).filter(search=query)
        return render(request, 'blog/post/search.html',
        {'form': form,
        'query': query,
        'results': results})

        ```
    - Buat blog/post/search.html
        ```
        {% extends "blog/base.html" %}
        {% load blog_tags %}

        {% block title %}Search{% endblock %}

        {% block content %}
            {% if query %}
                <h1>Posts containing "{{ query }}"</h1>
                <h3>
                    {% with results.count as total_results %}
                    Found {{ total_results }} result{{ total_results|pluralize }}
                    {% endwith %}
                </h3>
                {% for post in results %}
                <h4><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></h4>
                    {{ post.body|markdown|truncatewords_html:5 }}
                {% empty %}
                    <p>There are no results for your query.</p>
                {% endfor %}
                <p><a href="{% url 'blog:post_search' %}">Search again</a></p>
            {% else %}
                <h1>Search for posts</h1>
                <form method="get">
                    {{ form.as_p }}
                    <input type="submit" value="Search">
                </form>
            {% endif %}
        {% endblock %}
        ```
    - Edit urls.py
        - `path('search/', views.post_search, name='post_search'),`
    - http://127.0.0.1:8000/blog/search/

Stemming and ranking results 88
Weighting queries 89
Searching with trigram similarity 90
Other full-text search engines 91
Summary 91
