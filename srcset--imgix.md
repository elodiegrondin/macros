# Macro srcset #1 for Craft CMS

**Note #1:** In devMode, no srcset to avoid pending tasks caused by images transforms.

**Note #2:** In prodMode, images transforms by imgix.

## Macro

```
{% macro srcsetImgix(settings) %}
	{% spaceless %}

	 {# -- SETUP -- #}
		{% set class = (settings.class is defined and settings.class|length) ? settings.class : '' %}
		{% set lazyload = (settings.lazyload is defined and settings.lazyload == false) ? '' : 'lazyload' %}
		{% set paramsImgixFp = '&fp-x' ~ settings.asset.focalPoint.x ~ '&fp-y' ~ settings.asset.focalPoint.y %}
		{% set paramsCraftDefault = { mode: 'crop', position: 'center-center', quality: 40 } %}
		{% set paramsImgixDefault = '&fit=crop&crop=focalpoint&auto=format,compress&q=40' %}
		{% set sizes = (settings.sizes is defined and settings.sizes|length) ? settings.sizes : 'auto' %}
		{% set alt = (settings.alt is defined and settings.alt|length) ? settings.alt : (settings.asset is defined) ? settings.asset.title : '' %}
		{% set mode = (settings.asset.width > settings.asset.height) ? 'landscape' : 'portrait' %}

		{% if craft.app.config.general.imgix == false or settings.asset.extension == 'svg' %}

			{% set img = (settings.asset|length) ? settings.asset : '' %}
			{% set paramsCraft = (settings.paramsCraft is defined and settings.paramsCraft|length) ? settings.paramsCraft : paramsCraftDefault %}
	    {% set srcset = [] %}

		  {%- for outputDim in settings.srcset -%}
		    {%- if outputDim.width <= img.width -%}
		      {%- set srcset = srcset | merge([img.getUrl({ 
		        width: outputDim.width, 
		        height: (outputDim.height is defined) ? outputDim.height : 'auto',
		        mode: paramsCraft.mode,
		        position: paramsCraft.position,
		        quality: paramsCraft.quality
		       }) ~ ' ' ~ outputDim.width ~ 'w']) -%}
		    {%- endif -%}
		  {%- endfor -%}

		{% else %}

			{% set img = (settings.asset|length) ? craft.imgix.transformImage( settings.asset, { width: settings.width, height: settings.height }) : '' %}
			{% set srcset = (settings.srcset|length) ? craft.imgix.transformImage( settings.asset, settings.srcset) : '' %}
			{% set paramsImgix = (settings.paramsImgix is defined and settings.paramsImgix|length) ? settings.paramsImgix ~ paramsImgixFp : paramsImgixDefault ~ paramsImgixFp %}
			{% set dataSrcset = [] %}
			{% for item in srcset.transformed %}
				{% set dataSrcset = dataSrcset|merge([
					item.url ~ paramsImgix ~ ' ' ~ item.width ~ 'w'
				]) %}
			{% endfor %}

		{% endif %}

		{# -- OUTPUT IMG TAG -- #}
		{% if craft.app.config.general.imgix == false or settings.asset.extension == 'svg' %}

			<img class="fade {{ mode }} {{ lazyload ~ ' ' ~ class }}" 
			src="{{ img.getUrl({width: settings.width, height: settings.height, mode: paramsCraft.mode, position: paramsCraft.position, quality: paramsCraft.quality}) }}" 
			{{ (lazyload == true) ? 'data-' : '' }}src="{{ img.getUrl({width: settings.width, height: settings.height, mode: paramsCraft.mode, position: paramsCraft.position, quality: paramsCraft.quality}) }}" 
			{{ (lazyload == true) ? 'data-' : '' }}srcset="{{- srcset|join(', ') -}}"
			{{ (lazyload == true) ? 'data-' : '' }}sizes="{{ sizes }}" 
			alt="{{ alt }}">

		{% else  %}

			<img class="fade {{ mode }} {{ lazyload ~ ' ' ~ class }}" 
			src="{{ img.getUrl() ~ paramsImgix }}" 
			{{ (lazyload == true) ? 'data-' : '' }}src="{{ img.getUrl() ~ paramsImgix }}" 
			{{ (lazyload == true) ? 'data-' : '' }}srcset="{{ dataSrcset|join(', ') }}" 
			{{ (lazyload == true) ? 'data-' : '' }}sizes="{{ sizes }}" 
			alt="{{ alt }}">

		{% endif %}

	{% endspaceless %}
{% endmacro %}
```

## Basic usage
```
{% set settings = {
	asset: img,
	width: 768,
	height: 768,
	srcset: [
	 	{width: 768, height: 768},
	],
} %}

{% import '_macros/images' as macrosImages %}
{{ macrosImages.srcsetImgix(settings) }}
```



## All settings

**class:** (string) --optional | "" --default | add custom class name

**lazyload:** (bool) --optional | true --default | enable lazyload

**asset:** (asset) --required | craft AssetFileModel

**width:** (int) --required | image width for srcset fallback

**height:** (int) --required | image height for srcset fallback

**srcset:** (array) --required | responsive image dimensions

**paramsCraft:** (object) --optional | { mode: 'crop', position: 'center-center', quality: 65 } --default | craftCms transforms images params

**paramsImgix:** (string) --optional | '&fit=crop&crop=entropy&auto=enhance,compress&q=40' --default | imgix transforms images params

**sizes:** (string) --optional | 'auto' --default | sizes for srcset

**alt:** (string) --optional | settings.asset.title --default | image alternative text


## Authors

- **Élodie Grondin** - Everything is Fun - cropmark.


## Changelog

### 0.0.1 - 2017-11-10

#### Added

- Initial release.
