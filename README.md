# Live Project

## Introduction
For the last two weeks of my time at the Tech Academy, I worked with my peers as a team on a project using the Django framework. This project is an interactive website for managing one's collections of things related to various hobbies, as well as API and Data Scraped content for those hobbies. Working with a framework such as Django has allowed me to quickly create templates and functionality while keeping everything clean and organized. Utilizing branches, communicating as a team, getting assigned user stories, and having the opportunity to quickly learn [BeautifulSoup](#BeautifulSoup)/[API](#API) data scraping was a great experience to get a feel for what a developer may experience on a day-to-day basis.

Below are descriptions of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.


## Stories 1-5
* [Story 1](#story-1)
* [Story 2-3](#story-2-3)
* [Story 4-5](#story-4-5)



### Story 1:
For this first quick story, we were tasked with creating a new project app folder and to setup the essentials for our website (i.e. the templates, views, urls). I decided to pick a comic book website that implemented CRUD functionality so in my first user task, I made the base and home html templates, and added the views function.

My base template:

	{% load static %}
	<!DOCTYPE html>
	<html lang="en">
	   <head>
	      <meta charset="UTF-8">
	      <meta name="viewport" content="width=device-width, initial-scale=1">
	      <link rel="stylesheet" type="text/css" href="{% static 'css/comicbook_layout.css' %}">
	      <title>{% block title %}Comic Collectors{% endblock %}</title>
	   </head>
	   <body>
	   <!-- Navbar -->
	     <header>
	       <nav id="nav_container">
	         <ul>
                      <li class="url_inline"><a href="{% url 'comic_home' %}">Home</a></li>
                      <li class="url_inline"><a href="{% url 'comic_create' %}">Create</a></li>
                      <li class="url_inline"><a href="{% url 'comic_read' %}">Read</a></li>
                      <li class="url_inline"><a href="{% url 'comic_bs' %}">BeautifulSoup</a></li>
                      <li class="url_inline"><a href="{% url 'comic_api' %}">API</a></li>
	         </ul>
	       </nav>
	      </header>

	   <!-- Main -->
	   <main>
	     {% block content %}
	     {% endblock %}
	   </main>

	   <!-- Footer -->
	     <footer id="footer_container">
	       <p id="footer_text">&copy; 2023 | Comic Collectors</p
	     </footer>
	   </body>
	</html>

My Views Function:

	def comic_home(request):
	   return render(request, 'ComicBook/comic_home.html')


### Story 2-3:
Story 2 required us to create a model + form for the comic book and to add create functionality inside our views. Story 3 made us display those comic book details for read functionality.

The Model:

	from django.db import models

	class ComicBook(models.Model):
	   title = models.CharField(max_length=100)
	   publisher = models.CharField(max_length=50)
	   publication_date = models.CharField(max_length=25)
	   volume = models.IntegerField()
	   issue = models.IntegerField()

	   Comics = models.Manager()

	   def __str__(self):
	      return self.title

The Form:

	from django.forms import ModelForm
	from .models import ComicBook

	class ComicBookForm(ModelForm):
	   class Meta:
	     model = ComicBook
	     fields = '__all__'

The views functions:

	def comic_create(request):
	   form = ComicBookForm(data=request.POST or None)
	   if request.method == 'POST':
	      if form.is_valid():
	         form.save()
	         return redirect('../read')
	   content = {'form': form}
	   return render(request, 'ComicBook/comic_create.html', content)

	def comic_read(request):
	  entry = ComicBook.Comics.all()
	  content = {'entry': entry}
	  return render(request, 'ComicBook/comic_read.html', content)


### Story 4-5:
Finishing up on implementing our CRUD process, these two stories focused on creating update and delete functions.

	def comic_update(request, pk):
	   entry = get_object_or_404(ComicBook, pk=pk)
	   form = ComicBookForm(data=request.POST or None, instance=entry)
	   if request.method == 'POST':
	      if form.is_valid():
                 form.save()
                 return redirect('../../read')
	   content = {'form': form, 'entry': entry}
	   return render(request, 'ComicBook/comic_update.html', content)

	def comic_delete(request, pk):
	   entry = get_object_or_404(ComicBook, pk=pk)
	   if request.method == 'POST':
	      entry.delete()
	      return redirect('../../read')
	   content = {'entry': entry}
	   return render(request, 'ComicBook/comic_delete.html', content)


## Data Scraping Stories:
* [BeautifulSoup](#BeautifulSoup)
* [API](#API)



### BeautifulSoup:
For these next few stories, we were tasked with learning an easy-to-learn python library called BeautifulSoup that allows one to scrape data and parse through HTML/XML documents.

My BeautifulSoup template:

	{% extends "comic_base.html" %}

	{% block title %}BeautifulSoup{% endblock %}

	{% block content %}
	  <div id="bs_container">
            <h1>Midtown Comics</h1>
            <h4>Scraped from: <a target="_blank" href="https://digitalcomicmuseum.com/">"https://digitalcomicmuseum.com/"</a></h4>
            <p>{{ info }}</p>
	  </div>
	{% endblock %}

My BeautifulSoup function:

	def comic_bs(request):
	  page = requests.get("https://digitalcomicmuseum.com/")
    	  soup = BeautifulSoup(page.content, 'html.parser')
    	  tr_elements = soup.find_all("tr", class_="mainrow")
    	  info = ''

    	  for tr in tr_elements:
	    td_elements = tr.find_all("td")
	    if len(td_elements) > 1:
              second_td = td_elements[1]
              text = second_td.get_text().strip()
              if not text.startswith('Latest Download:'):
                info += text + '\n'

	  content = {'info': info}
	  return render(request, 'ComicBook/comic_bs.html', content)


### API:
After the BeautifulSoup story, we were charged with the task of connecting to our chosen API and getting the JSON response, parsing through the JSON file returned and displaying the information.

My API template:

	{% extends "comic_base.html" %}

	{% block title %}Marvel Comics API{% endblock %}

	{% block content %}
	   <div id="api_container">
	      <h1>Marvel Comics API Response</h1>

	      <h2>Comic Books</h2>
	      <ul>
	         {% for comic in api_data.data.results %}
		    <li>
		       <strong>Title:</strong> {{ comic.title }}<br>				<strong>Description:</strong> {{ comic.description }}<br>
			<strong>Price:</strong> ${{ comic.prices.0.price }}<br>
		    </li>
                    <br>
              	{% endfor %}
	     </ul>
	  </div>
	{% endblock %}

my API function:

	def comic_api(request):
	   # API authentication keys
	   public_key = 'b7dce7b253b4abfc69250e57413bdc49'
	   private_key = '7f49c7fed8b25d7695a3378e9027c2a515cdf5ed'

	   # Generate a timestamp and hash
	   timestamp = str(int(time.time()))
	   hash_input = f'{timestamp}{private_key}{public_key}'.encode('utf-8')
	   hash_value = hashlib.md5(hash_input).hexdigest()

	   # Build the API request URL
	   base_url = 'https://gateway.marvel.com/v1/public/comics'
	   querystring = {
	      'apikey': public_key,
	      'ts': timestamp,
	      'hash': hash_value,
	   }

	   # Make the API request
	   response = requests.get(base_url, params=querystring)

	   # Check if the request was successful
	   if response.status_code == 200:
	      api_data = response.json()

	   # Pass the JSON data to your API template
	   context = {
	      'api_data': api_data
	   }
	   return render(request, 'ComicBook/comic_api.html', context)





