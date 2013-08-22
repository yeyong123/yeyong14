---
layout: post
title:  "面包屑"
date:   2013-08-22 10:51:42
categories: rails
---
为了索引方便，加上了面包屑功能

首先安装gem
`gem 'simple-navigation'`
在执行
`rails generate navigation_config`
接着配置文件

`config/navigation.rb`

```ruby
navigation.items do |primary|
# Add an item to the primary navigation. The following params apply:
# key - a symbol which uniquely defines your navigation item in the scope of the primary_navigation
# name - will be displayed in the rendered navigation. This can also be a call to your I18n-framework.
# url - the address that the generated item links to. You can also use url_helpers (named routes, restful routes helper, url_for etc.)
# options - can be used to specify attributes that will be included in the rendered navigation item (e.g. id, class etc.)
#           some special options that can be set:
#           :if - Specifies a proc to call to determine if the item should
#                 be rendered (e.g. <tt>:if => Proc.new { current_user.admin? }</tt>). The
#                 proc should evaluate to a true or false value and is evaluated in the context of the view.
#           :unless - Specifies a proc to call to determine if the item should not
#                     be rendered (e.g. <tt>:unless => Proc.new { current_user.admin? }</tt>). The
#                     proc should evaluate to a true or false value and is evaluated in the context of the view.
#           :method - Specifies the http-method for the generated link - default is :get.
#           :highlights_on - if autohighlighting is turned off and/or you want to explicitly specify
#                            when the item should be highlighted, you can set a regexp which is matched
#                            against the current URI.  You may also use a proc, or the symbol <tt>:subpath</tt>. 
#
primary.item :home, '首页', home_path, :highlights_on => /(^\/$|^\/home)/ do |home|
	home.item :products, '产品中心', products_path
		Category.all.each do |cate|
			home.item :cate, cate.name, category_path(cate.id) do |category|
				cate.tags.each do |tag|
					category.item :tag, tag.name, tag_path(t:tag) do |tags|
						tag.lists.each do |list|
							tags.item :list, list.name, products_path(t:list) do |lists|
								list.products.each do |product|
									lists.item :product, product.title, product_path(product.id)
							end
						end
					end
				end
			end
		end
	end

	home.item :abouts, 'abouts', abouts_path
	home.item :companies, 'companies', companies_path
end
# Add an item which has a sub navigation (same params, but with block)

# You can also specify a condition-proc that needs to be fullfilled to display an item.
# Conditions are part of the options. They are evaluated in the context of the views,
# thus you can use all the methods and vars you have available in the views.

# you can also specify a css id or class to attach to this particular level
# works for all levels of the menu
# primary.dom_id = 'menu-id'
# primary.dom_class = 'menu-class'

# You can turn off auto highlighting for a specific level
# primary.auto_highlight = false

	end

```
然后在`views` `layouts/appliation.html.erb`渲染

```ruby
<div id="navigation">
	<%= render_navigation(:renderer => :breadcrumbs, :join_with => ' &raquo; ') %>
</div>

```
##效果这样:    首页 >> 产品中心 >> 产品
