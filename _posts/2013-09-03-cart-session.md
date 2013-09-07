---
layout: post
title: "将购物车与当前用户关联"
description: "关联购物车"
category: "cart gem"
tags: ["cart"]
---
{% include JB/setup %}
当用户添加进购物车也是可以加进去的。
当用户退出之后，购物车的内容也随着用户的退出被清空了
但是数据库的内容依然有的
好了下面就是我的定义的，现在不会随着用户的退出而清空购物车了


`rails g migration add_user_id_to_carts user_id:integer`

```ruby
class Cart < ActiveRecord::Base
	has_many :line_items, dependent: :destroy
	belongs_to user
end

class LineItem < ActiveRecord::Base
	attr_accsessible :product_id, :cart_id, :product
	belongs_to :product
	belongs_to :cart
end
```

`cart`表中的是空的，只有时间戳

`ApplicationConteroller`

```ruby
class ApplicationControlleer < ActiveController:Base
	before_filter :current_user
	
	private

	def current_cart
	#下面的代码有很大的重构的空间
	#现在先用着的，等我想好了在重构。要是不写注释的话
	#可能明天就看不懂
		if user_signed_in? && current_user #首先先检查用户有没有登录
			if current_user.cart.nil?				 #用户已经登录但在数据库没有找到与这个用户相关联的购物车。
				Cart.find(session[:cart_id])   # 就为这个用户创建一个ID
			else
				Cart.find_by_user_id(session[:user_id]) # 否则就找到这个购物车中用户的ID。
			end
		end
		unless user_signed_in?								#如果用户没有登录
			Cart.find(session[:cart_id])				#就为这个访客建立一个会话
		end
		rescue ActiveRecord::RecordNotFound		#如果这个临时的会话没有找到就拦截这个没有会话的错误。
		if user_signed_in? && current_user		#这个是判断语句，判断用户登录之后没有找到用户购物车的拦截错误语句，新用户没有
			@cart = current_user.create_cart		#找到就建立一个新的用户购物车
		else
			@cart = Cart.create									#判断没有登录的用户的创建购物车
		end
		session[:cart_id] = @cart.id					#将创建购物车ID，赋值给会话中
		@cart
	end
end
```
控制器中的`carts`, `line_items`

`controller/carts_controller.rb`

```ruby
class CartsController < ApplicationController

	def show
	  if user_signed_in? && current_user #用户登录的显示内容
			@cart = current_user.cart				 #显示当前用户的购物车内容
		else
			@cart = Cart.find(!current_user) #访客显示的就是非登录用户所有数据
		end
	end

end

class LineItemsController < ApplicationController

	def create
		if user_signed_in? && current_user
			@cart = current_user.cart
		else
			@cart = current_cart
		end
		product = Product.find(params[:product_id])
		@line_item = @cart.line_items.build(product: product)
		if @line_item.save
			redirect_to @line_item.cart
		end
	end
end
```
最后路由定义下

```ruby
resource :cart
```

Views里面添加

```erb
<%= link_to 'Add cart', line_items_path(product_id: product)  %>
```
显示cart的内容

```erb
<table class="table">
<% @cart.line_items.each do |item| %>
<tr>
<td><%= item.product.title %></td>
</tr>
<% end %>
</table>
```
