---
layout: post
title: "railsjQueryFileUpload模型嵌套使用"
description: "jquery-fileupload"
category: "jQuery"
tags: ["gem"]
---
{% include JB/setup %}

最近在做一个网站， 要一次上传多张图片，
发现有一个`gem jquery-fileupload-rails`,这个模型对单一模型很好用，
但在模型嵌套的时候就要配置一下
这个是我的配置，写下来做个笔记，下次遇到在做参考
首先`Gemfile`定义`gem`

```ruby
gem `jquery-fileupload-rails`
gem `carrierwave` #上传图片用的
gem `mini_magick`
```
首先先定义上传图片
`rails g uploader image`
到`app/uploaders/image_uploader.rb`简单的定义

```ruby
class ImageUploader < Carrierwave::Uploader::Base
	include Carrierwave::MiniMagick
#
#
#
	version :thumb do
		process :resize_to_limit => [100, 100]
	end
```

有二个模型

```ruby
class Picture < ActiveRecord::Base
	attr_accessible :image, :image_cache, :product_id
	belongs_to :product
	include Rails.application.routes.url_helpers
	mount_uploader :image, ImageUpload

	def to_jq_upload
		{
			"name" => filename, #这里的要另外的改下。
			"size" => image.size,
			"url"  => image.url,
			"thumbnail_url" => image.thumb.url,
			"delete_url" => picture_path(id: id),
			"delete_type" => "DELETE"
		}
	end
end

#product的模型

class Product < ActiveRecord::Base
	attr_accessible :title, :content,:pictures_attributes 
	has_many :pictures, dependent: :destroy

	accepts_nested_attributes_for :pictures
end
```
接着完成控制器的嵌套，当初就卡在控制器这里

```ruby
class PicturesController < ApplicationController

	#def index
	#end
	
	def create
		@picture = Picture.new(params[:picture])
		if @picture.save
		respond_to do |format|
		format.html {
			render :json => [@picture.to_ja_upload].to_json
		}
		format.json {
			render :json => [@picture.to_jq_upload].to_json
		}
		end
		else
		render :json => [{:error=> "custom_failure"}], :status => 304
		end
	end

class ProductsController < ApplicationController
	
	def index
	end

	def new
		@product = Product.new
		@product.pictures.build
	end

	def create
		@product = Product.new(params[:product])
		#这里是picture关联product。为了能上传多张的图片，所以这里的pictures是复数
		#先找出pictures里面product.id，如果有的话， 赋值给@pictures的变量中。
		#在将@pictures 传递给关联的product中。这样就将多张图片上传到一个procuct.id中
		@pictures = Picture.where(product_id: @product.id)
		@product.pictures << @pictrues
		respond_to do |format|
		if @product.save
			format.html {redirect_to @product, notice: 'Product was successfully'}
			format.json {render json: @product, status: create, location: @product}
		else
			format.html { render action: "new"}
			format.json {render json: @product.errors, status: "fail"}
		end
	end
end
```
视图中的就是简单的定义一下

```erb
<%= form_for (@product), do |f| %>
<%= f.label :title %>
<%= f.text_field :title %><br />
<%= f.label :content %>
<%= f.text_field :content %>
<%= f.submit %>
<% end %>
<%= form_for @picture do |pic| %>
	<div class="row fileupload-buttonbar">
		<div class="span7">
			<!-- The fileinput-button span is used to style the file input field as button -->
			<span class="btn btn-success fileinput-button">
				<i class="icon-plus icon-white"></i>
				<span>Add files...</span>
							<%= pic.file_field :image %>
			</span>
			<button type="submit" class="btn btn-primary start">
				<i class="icon-upload icon-white"></i>
				<span>Start upload</span>
			</button>
			<button type="reset" class="btn btn-warning cancel">
				<i class="icon-ban-circle icon-white"></i>
				<span>Cancel upload</span>
			</button>
			<button type="button" class="btn btn-danger delete">
				<i class="icon-trash icon-white"></i>
				<span>Delete</span>
			</button>
			<input type="checkbox" class="toggle">
		</div>
		<div class="span5">
			<!-- The global progress bar -->
			<div class="progress progress-success progress-striped active fade">
				<div class="bar" style="width:0%;"></div>
			</div>
		</div>
	</div>
	<!-- The loading indicator is shown during image processing -->
	<div class="fileupload-loading"></div>
	<br>
	<!-- The table listing the files available for upload/download -->
	<table class="table table-striped"><tbody class="files" data-toggle="modal-gallery" data-target="#modal-gallery"></tbody>
	</table>
	<% end %>
<script>
	var fileUploadErrors = {
	maxFileSize: 'File is too big',
	minFileSize: 'File is too small',
	acceptFileTypes: 'Filetype not allowed',
	maxNumberOfFiles: 'Max number of files exceeded',
	uploadedBytes: 'Uploaded bytes exceed file size',
	}
	</script>

<!-- The template to display files available for upload -->
<script id="template-upload" type="text/x-tmpl">
	{% for (var i=0, file; file=o.files[i]; i++) { %}
	<tr class="template-upload fade">
		<td class="preview"><span class="fade"></span></td>
		<td class="name"><span>{%=file.name%}</span></td>
		<td class="size"><span>{%=o.formatFileSize(file.size)%}</span></td>
		{% if (file.error) { %}
		<td class="error" colspan="2"><span class="label label-important">{%=locale.fileupload.error%}</span> {%=locale.fileupload.errors[file.error] || file.error%}</td>
		{% } else if (o.files.valid && !i) { %}
		<td>
			<div class="progress progress-success progress-striped active"><div class="bar" style="width:0%;"></div></div>
		</td>
		<td class="start">{% if (!o.options.autoUpload) { %}
			<button class="btn btn-primary">
				<i class="icon-upload icon-white"></i>
				<span>{%=locale.fileupload.start%}</span>
			</button>
			{% } %}</td>
		{% } else { %}
		<td colspan="2"></td>
		{% } %}
		<td class="cancel">{% if (!i) { %}
			<button class="btn btn-warning">
				<i class="icon-ban-circle icon-white"></i>
				<span>{%=locale.fileupload.cancel%}</span>
			</button>
			{% } %}</td>
	</tr>
	{% } %}
</script>
<!-- The template to display files available for download -->
<script id="template-download" type="text/x-tmpl">
	{% for (var i=0, file; file=o.files[i]; i++) { %}
		<tr class="template-download fade">
			{% if (file.error) { %}
				<td></td>
				<td class="name"><span>{%=file.name%}</span></td>
				<td class="size"><span>{%=o.formatFileSize(file.size)%}</span></td>
				<td class="error" colspan="2"><span class="label label-important">{%=locale.fileupload.error%}</span> {%=locale.fileupload.errors[file.error] || file.error%}</td>
				{% } else { %}
				<td class="preview">{% if (file.thumbnail_url) { %}
					<a href="{%=file.url%}" title="{%=file.name%}" rel="gallery" download="{%=file.name%}"><img src="{%=file.thumbnail_url%}"></a>
					{% } %}</td>
				<td class="name">
					<a href="{%=file.url%}" title="{%=file.name%}" rel="{%=file.thumbnail_url&&'gallery'%}" download="{%=file.name%}">{%=file.name%}</a>
				</td>
				<td class="size"><span>{%=o.formatFileSize(file.size)%}</span></td>
				<td colspan="2"></td>
				{% } %}
			<td class="delete">
				<button class="btn btn-danger" data-type="{%=file.delete_type%}" data-url="{%=file.delete_url%}">
					<i class="icon-trash icon-white"></i>
					<span>{%=locale.fileupload.destroy%}</span>
				</button>
				<input type="checkbox" name="delete" value="1">
			</td>
		</tr>
		{% } %}
</script>



<!-- The XDomainRequest Transport is included for cross-domain file deletion for IE8+ -->
<!--[if gte IE 8]><%= javascript_include_tag "jquery.xdr-transport.js" %><![endif]-->


<script type="text/javascript" charset="utf-8">
	$(function () {
			// Initialize the jQuery File Upload widget:
			$('#fileupload').fileupload();
			// 
			// Load existing files:
			$.getJSON($('#fileupload').prop('action'), function (files) {
				var fu = $('#fileupload').data('fileupload'), 
				template;
				fu._adjustMaxNumberOfFiles(-files.length);
				template = fu._renderDownload(files)
				.appendTo($('#fileupload .files'));
				// Force reflow:
				fu._reflow = fu._transition && template.length &&
				template[0].offsetWidth;
				template.addClass('in');
				$('#loading').remove();
				});

			});
</script>
```

