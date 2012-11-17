# วิธีการใช้งาน Deferred 

## ติดตั้ง
	npm install Deferred

## ปัญหา CallBack ( Callback Hell )

การที่มี callback มากๆทำให้เราสับสนกับ tag ปิดของ callback ของแต่ละชุดได้ 

	ws_db.listCategory(perPage, page, function(docCategories){
		ws_db.listProduct(perPage, page, function(docProduct){
			ws_db.listMember(perPage, page, function(docMember){
				ws_db.listEmployee(perPage, page, function(docEmployee){
					
					var data = {
						categories: docCategories,
						products: docProduct,
						members: docMember,
						members: docEmployee
					};
						
					res.render('main', data);
				});
			});
		});
	});
	

## การนำ Deferred มาใช้

ตัวอย่างการใช้งานกับ mongojs

	var databaseURL = 'localhost/mydb';
	var collections = ['product','category','member'];
	
	var mongojs = require('mongojs');
	var db = mongojs.connect(databaseURL, collections);

	var express = require('express')
	  , http = require('http')
	  , path = require('path');
	
	function getTable(collection){
		var deferred = Deferred();
		collection.find(function(err, doc){
			if (err) deferred.reject(err);
			deferred.resolve(doc);
		}).sort({_id:1});	
		return deferred.promise()
	}

	var app = express();

	app.configure(function(){
	  app.set('port', process.env.PORT || 3000);
	  app.set('views', __dirname + '/views');
	  app.set('view engine', 'jade');
	  app.use(express.favicon());
	  app.use(express.logger('dev'));
	  app.use(express.bodyParser());
	  app.use(express.methodOverride());
	  app.use(express.cookieParser('keyboard rat'));
	  app.use(express.session());
	  app.use(app.router);
	  app.use(express.static(path.join(__dirname, 'public')));
	});

	app.get('/', function(req, res){
		var deferred = Deferred();
		var categories = getTable(db.category);
		var product = getTable(db.product);
		var member = getTable(db.member);
		
		var lookup = Deferred.when(categories, product, member);
		lookup.done(function(categories, product, member){
			res.render('home', {
				'categories': categories,
				'product': product,
				'member': member
			});
		}).fail(function(err) {
			res.render('error', {'error': err})
		});
	});
	
	http.createServer(app).listen(app.get('port'), function(){
	  console.log("server listening on port " + app.get('port'));
	});
	
## จุดสำคัญ

deferred มีจุดสำคัญง่ายๆ 
- ประกาศก่อน var deferred = Deferred();
- เช็ค err ด้วย if (err) deferred.reject(err);
- ได้ข้อมูลที่ต้องการแล้วก็ใช้ deferred.resolve(data);
- จบ block นึงแล้วก็ deferred.promise(); 

	function getTable(collection){
		var deferred = Deferred();
		collection.find(function(err, doc){
			if (err) deferred.reject(err);
			deferred.resolve(doc);
		}).sort({_id:1});	
		return deferred.promise();
	}

## อ้างอิง

ได้ตัวอย่างความรู้นี้มาจาก <http://davidwalsh.name/deferred>