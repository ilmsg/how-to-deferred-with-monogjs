# วิธีการใช้งาน Deferred 

## ติดตั้ง
	npm install Deferred

## การใช้งาน

ตัวอย่างการใช้งานกับ mongodb 
	var mongojs = require('mongojs');
	var db = mongojs.connect('locahost/shopmini', ['category','product']);

var Express = require('express');
var Deferred = require('Deferred');

app = Express();
app.get('/', function(req, res){
   var deferred = Deferred();
   var cat = getCategory();
   var pro = getProduct();

   var lookup = Deferred.when(cat , pro);
   lookup.done(function(categories, product){
      console.log( require('util').inspect(categories) );
      console.log( require('util').inspect(product) );
   }).fail(function(err) {
      console.log( err );
   });
});

function getCategory(){
   var deferred = Deferred();
   db.category.find(function(err, doc){
      deferred.resolve(doc);
   }).sort({_id:1});	
   return deferred.promise();
}

function getProduct(){
   var deferred = Deferred();
   db.product.find(function(err, doc){
      deferred.resolve(doc);
   }).sort({_id:1});	
   return deferred.promise();
}