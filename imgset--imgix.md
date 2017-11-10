# Macro imgset--imgix #1 for Craft CMS

**Note #1:** In devMode, no srcset to avoid pending tasks caused by images transforms.

**Note #2:** In prodMode, images transforms by imgix.

## Prerequisites

- [imgix plugin](https://superbig.co/plugins/imgix) by Superbig


## Macro

	{% macro imgsetImgix(settings) %}
		{% spaceless %}
			
			{# -- SETUP -- #}
			{% set class = (settings.class is defined and settings.class|length) ? settings.class : '' %}
			{% set asset = (settings.asset|length) ? settings.asset : '' %}
			{% set width = (settings.width is defined and settings.width|length) ? settings.width : 'auto' %}
			{% set height = (settings.height is defined and settings.height|length) ? settings.height : 'auto' %}
			{% set imgset = [
				{ width: width, height: height },
				{ width: (width matches '/^\\d+$/') ? (width * 2) : width, height: (height matches '/^\\d+$/') ? (height * 2) : height },
				{ width: (width matches '/^\\d+$/') ? (width * 3) : width, height: (height matches '/^\\d+$/') ? (height * 3) : height }
			] %}
			{% set img = craft.imgix.transformImage(asset, imgset) %}
			{% set paramsCraftDefault = { mode: 'crop', position: 'center-center', quality: 65 } %}
			{% set paramsImgixDefault = '&fit=crop&crop=entropy&auto=enhance,compress&q=40' %}


			{# -- OUTPUT BGIMG TAG -- #}
			{% if craft.config.devMode == true %}

				{% set paramsCraft = (settings.paramsCraft is defined and settings.paramsCraft|length) ? settings.paramsCraft : paramsCraftDefault %}

				<div class="{{ class }}" style="background-image: url({{ asset.getUrl({width: width, height: height, mode: paramsCraft.mode, position: paramsCraft.position, quality: paramsCraft.quality}) }});"></div>

			{% else  %}

				{% set paramsImgix = (settings.paramsImgix is defined and settings.paramsImgix|length) ? settings.paramsImgix : paramsImgixDefault %}

				<div class="{{ class }}" style="background-image: url({{ img.transformed[0].url ~ paramsImgix }}); background-image: -webkit-image-set(url({{ img.transformed[0].url ~ paramsImgix }}) 1x, url({{ img.transformed[1].url ~ paramsImgix }}) 2x, url({{ img.transformed[2].url ~ paramsImgix }}) 3x); background-image: image-set({{ img.transformed[0].url ~ paramsImgix }} 1x, {{ img.transformed[1].url ~ paramsImgix }} 2x, {{ img.transformed[2].url ~ paramsImgix }} 3x);"></div>	

			{% endif %}

		{% endspaceless %}
	{% endmacro %}


## Basic usage

	{% set settings = {
		class: 'banner',
		asset: entry.banner.first,
		height: 300,
	} %}

	{% import '_macros/imgset--imgix' as macrosBgImg %}

	{{ macrosBgImg.imgsetImgix(settings) }}



## All settings

| Option | Type | Default value | Flag | Description |
| ------ | ---- | ------------- | ---- | ----------- |
| **class** | string | "" | --optional | add custom class name |
| **asset** | asset |  | --required | craft AssetFileModel |
| **width** | int or 'auto' | 'auto' | --optional  | image width for imgset fallback |
| **height** | int or 'auto' | 'auto' | --optional  | image height for imgset fallback |
| **paramsCraft** | object | { mode: 'crop', position: 'center-center', quality: 65 } | --optional  | craftCms transforms images params |
| **paramsImgix** | string | '&fit=crop&crop=entropy&auto=enhance,compress&q=40' | --optional  | imgix transforms images params |


## Authors

- **Élodie Grondin** - Everything is Fun - cropmark.


## Changelog

### 0.0.1 - 2017-11-10

#### Added

- Initial release.
