[Polymorphic](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations) associations are used to add refence to something that is shared, for example products and users has many pictures so pictures has  imageable_id (instead of product_id or user_id) and imageable_type: string ('product','user').


[STI](http://eewang.github.io/blog/2013/03/12/how-and-when-to-use-single-table-inheritance-in-rails/) 
http://enterpriserails.chak.org/full-text/chapter-10-multiple-table-inheritance
http://thibaultdenizet.com/tutorial/single-table-inheritance-with-rails-4-part-1/

STI maps all fields into a single table http://martinfowler.com/eaaCatalog/classTableInheritance.html
CTI maps each clas into its own table http://martinfowler.com/eaaCatalog/classTableInheritance.html
