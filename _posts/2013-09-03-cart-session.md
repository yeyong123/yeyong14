---
layout: post
title: "将购物车与当前关联"
description: "关联购物车"
category: "cart gem"
tags: ["cart"]
---
{% include JB/setup %}

按照rails敏捷开发的步骤定义了购物车
购物车的功能的是实现了。
但是加了`devise`的验证用户之后，
当用户添加进购物车也是可以加进去的。
当用户退出之后，购物车的内容也随着用户的退出被清空了
但是数据库的内容依然有的
后来上github上面找了一个gem.
`acts_as_shopping_cart`
跟敏捷开发中的购物车的实现也是差不多的
好了下面就是我的定义的，现在不会随着用户的退出而清空购物车了

`gem 'acts_as_shopping_cart'`
首先先定义三个模型
`cart`, `cart_item`, `user`

先把`user_id`加到`cart`

`rails g migration add_user_id_to_carts user_id:integer`

```ruby
class Cart < ActiveRecord::Base
	acts_as_shopping_cart_using :cart_item
	belongs_to user
end

class CartItem < ActiveRecord::Base
	acts_as_shopping_cart_item_for :cart
end
```

`cart`表中的是空的，只有时间戳
关键是`cart_items`表

```ruby
create_table :cart_items do |t|
	t.shopping_cart_item_fields
end
```
下一步创建购物车

`controller/carts_controller.rb`

```ruby
class CartsController < ApplicationController
	before_filter :extract_cart

	def create
	@product = Product.find(params[:product_id])
	@cart.add(@product, @product.title)
	rediect_to cart_path

	def show
	end

	private

	def extract_cart
		cart_id = session[:cart_id]
		@cart = session[:cart_id] ? current_user.carts.find(cart_id) : current_user.carts.create
		session[:cart_id] = @cart.id
	end

#其中current_user.carts.find(cart_id) : current_user.carts.create
#是我另外加的，为的就是与当前用户关联起来。这样当用户加入购物车之后，在退出。
#上次加入的产品依然存在。

end

```
最后路由定义下

```ruby
resource :cart
```

Views里面添加

```ruby

<%= link_to 'Add cart', cart_path(product_id: product), method: :post %>
```

