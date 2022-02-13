# dango_bs_carousel
A bootstrap carousel with asynchronous image loading using javascript webworkers.  Because images are loaded asychronously, your user can continue to interact with the site.

## installation
to be confirmed

## usage
add `django-bs-carousel` to your installed apps section of your settings.py.
add `{% include 'django-bs-carousel/carousel.html' %} to the template that will display the carousel.
In your settings.py file add the following...
```
IMAGE_SIZE_LARGE = "1024x768"
IMAGE_SIZE_SMALL = "360x640"

NUM_IMAGES_PER_REQUEST = 5

CAROUSEL_RANDOMIZE_IMAGES = False
CAROUSEL_USE_CACHE = False
CAROUSEL_IMG_PAUSE = 6500
CAROUSEL_OFFSET = True

# image model must have a field called 'file' which is the image file
# and an autoincrement pk integer field named 'pk'.
DJANGO_BS_CAROUSEL_IMAGE_MODEL = "django_artisan.UserProductImage"
```
Note that whatever model you are using for the images, each record must have a field call file, that is the image_file field.

In the view that is associated with the template that displays the carousel, place the following context variables:
```python
    def get_context_data(self, **kwargs) -> dict:
        context = super().get_context_data(**kwargs)
        context['images'] = (UserProductImage.objects
                              .filter(active=True))
        context['randomize_images'] = conf.settings.CAROUSEL_RANDOMIZE_IMAGES
        context['use_cache'] = conf.settings.CAROUSEL_USE_CACHE
        context['offset'] = conf.settings.CAROUSEL_OFFSET
        context['loading_image'] = 'django-bs-carousel/images/spinning-circles.svg'
        context['image_size_large'] = conf.settings.IMAGE_SIZE_LARGE
        context['image_size_small'] = conf.settings.IMAGE_SIZE_SMALL
        context['images_per_request'] = conf.settings.NUM_IMAGES_PER_REQUEST
        context['image_pause'] = conf.settings.CAROUSEL_IMG_PAUSE
        context['csrftoken'] = csrf.get_token(self.request)
        return context

```
Note that my UserProductImage model has a boolean field called 'active' which denotes that the image is available for display.
If you use the default template, then your Image model will need a field named 'caption'.

If you want to use a custom template, then simply copy the following into a new template file in your project:
```html
{% load static %}
    <div id="carousel-large-background" class="carousel slide carousel-fade" data-bs-interval="{{image_pause}}" data-bs-ride="carousel" data-bs-pause="false">
      <input type="hidden" name="csrfmiddlewaretoken" value="{{csrftoken}}">
      <input type="hidden" id="hidden-data" data-images-per-request="{{images_per_request}}" data-use-cache="{{use_cache}}" data-randomize-images="{{randomize_images}}" data-loading-image="{% static loading_image %}" data-image-size-large="{{image_size_large}}" data-image-size-small="{{image_size_small}}" data-offset="{{offset}}">
      <div class="carousel-inner">
        {% for image in images %}
          <div id="image-{{ forloop.counter }}-carousel" class="carousel-item">
            <img src="{% static loading_image %}" size="{{image_size}}" height="100%" class="carousel-image" id="{{image.id}}" data-image-src="{{image.file.url}}">
            <div class="carousel-caption p-2 col-sm-6 col-md-4 col-lg-3 d-md-block text-white">
              {{image.caption|safe}}
            </div>
          </div>
        {% endfor %}
      </div>
      {% if images.exists %}
        <a class="carousel-control-prev" href="#carousel-large-background" role="button" data-bs-slide="prev">
          <span class="carousel-control-prev-icon" aria-hidden="true"></span>
          <span class="visually-hidden">Previous</span>
        </a>
        <a class="carousel-control-next" href="#carousel-large-background" role="button" data-bs-slide="next">
          <span class="carousel-control-next-icon" aria-hidden="true"></span>
          <span class="visually-hidden">Next</span>
        </a>
      {% endif %}
    </div>
    {% block body_js %}
      <script src="{% static 'django-bs-carousel/js/carousel.js' %}" type="application/javascript" referrerpolicy="origin" defer=""></script>
    {% endblock body_js %}
```
Note that your base.html template can have a `{% block body_js %} {% endblock body_js %}` at the end of your content, so that the javascript 'carousel.js' can be loaded properly.  Also, you can change image.caption for any text field etc. rather than being bound to placing a field named 'caption' on your images model.

## options
The following options in your settings.py file control the way the carousel works.

### CAROUSEL_RANDOMIZE_IMAGES = False
This randomizes the presentation of the images
### CAROUSEL_USE_CACHE = False
If this is True, then the webworker will make a request to a view function on django-bs-carousel.  You will need to have sorl-thumbnail and PIL installed using pip.  The view function uses sorl-thumbnail to create two image thumbnails, one large and one small, at the sizes you set using the two options below, image_size_large and image_size_small.  These two image sizes are ideals for the presentation of images to either a desktop or a mobile.  The thumbnail operation converts it to webp, if the browser supports this, and then caches the images, so that they can be recovered later.  The webworker then loads the images asynchronously.  So, if you use the cache you can get webp images, which will offer the quality of webp to the user.  If you do not use the cache the images are loaded as they are found on the image file on your model.  You can change all the images to a particular format and size when they are uploaded.  Note that even if you set useCache to false, the browser will cache the webworker's loaded file, so as long as the user does not clear their browser cache they will get cached image loading.
### IMAGE_SIZE_LARGE = "1024x768"
The large size of image if useCache is set to True.
### IMAGE_SIZE_SMALL = "360x640"
The small size of image if useCache is set to True.
### NUM_IMAGES_PER_REQUEST = 5
This is the number of images that are loaded each time.  So, if useCache is set to True, the number of requests made will be number_of_images/NUM_IMAGES_PER_REQUEST.  If useCache is set to False, then the files will be loaded in sets of files numbering NUM_IMAGES_PER_REQUEST.  In either case the webworker loads the number of images per request into an ArrayBuffer before using content transfer to move the images back to the main thread, so there is a maximum useful number.  I set it to 5 and it seems to work fine.  I have yet to test it live though.
### CAROUSEL_IMG_PAUSE = 6500
This is the amount of time per image slide
### CAROUSEL_OFFSET = True
This loads the first image immediately.  Even when it is set it is replaced with the image as loaded by the script.  It just shows the standard image a little bit quicker.

### DJANGO_BS_CAROUSEL_IMAGE_MODEL = "django_artisan.UserProductImage"
This is the image model which must have a field called 'file' which is the image file.  The image model must have an autoincrement pk integer field named 'pk'.
The string value is the app name, and then the model from models.py or similar.

Enjoy!
