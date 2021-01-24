# Template of Oxford University personal page

This template serves as a good start to create your personal page. It includes easy-to-use markdown pages and automatic bibliography section using [jekyll-scholar](https://github.com/inukshuk/jekyll-scholar).


## Install
1. Simply clone the repository.

	```
	git clone https://github.com/oxfordcontrol/personal_homepage_template.git
	```


2. Then install the required ruby gems
	```
	bundle install
	```
3. Generate the website
	```
	bundle exec jekyll build
	```
   The website will be in the folder `_site`. Just copy it to your favorite provider.

To preview the website on your localhost just run
```
bundle exec jekyll serve
```
Your website will be [here](http://localhost:4000/).

## Editing the profile details
To edit the profile details simply change the `_config.yml` file in the root folder. These changes will be automatically reflected on the generated website.


## Pages
All the pages are under `src/_pages`. They are all mostly written in Markdown.

### Downloads
All the downloads should be put in the `/assets/downloads` folder and are directly accessible using Markdown links within the website pages. For examples, we can link a pdf file using `[slides](/assets/downloads/presentations/2017/example_presentation.pdf)`. 

### Bibliography
The bibliography page is directly created from the Bibtex file `src/_bibliography/bibliography.bib`.
The additional fields `abstract`, `url`, `pdf` and `code` are automatically extracted when generating the page with links and descriptions.

## Example webpage
An example website generated using this template can be found [here](https://bstellato.github.io/).

