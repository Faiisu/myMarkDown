
**It's not all commands, it's just the commands I've used.**
# 1. Pull query basic
```js
//basic .find() structure
db.prodicts.fin(
	{}, //for condition
	{} //for select fields
)

//pull all
db.products.find({})
//pull only catagory = sneakers
db.products.find({ category: "sneakers" })
//pull only have "limited edition" in array tags
db.products.find({ tags: "limited edition" })
//pull all with out category fields exist
db.collection.find({ category: { $exists: false } })

//nested document
//customer is document inside main document
//want to pull only name of customer = John Doe
db.orders.find({ "customer.name": "John Doe" }, {}) //empty {} can use for select fields

```
# 2. Condition Query
### 2.1 finding same value
```js
//find category == "sneakers"
db.products.find({ category: "sneakers" })

//find Array size = 3
db.collection.find({ myArray: { $size: 3 } })z z
```
### 2.2 <, <=, >, >=
```js
db.products.find({ price: { $gt: 100, $lt: 500 } }) // 100 < price < 500
db.products.find({ price: { $gte: 100, $lte: 500 } }) // 100 <=price <= 500
```
### 2.3 in, nin
```js
//brand is Nike or Adidas
db.products.find({ brand: { $in: ["Nike", "Adidas"] } }) 
//all except Puma and Reebok
db.products.find({ brand: { $nin: ["Puma", "Reebok"] } })
```
### 2.4 or, and
```js
//or
db.products.find({ 
  $or: [
    { category: "sneakers" }, 
    { price: { $lt: 200 } }
  ] 
})
//and
db.products.find({ 
  $and: [
    { category: "sneakers" }, 
    { price: { $gt: 300 } }
  ] 
})
```
# 3. select Fields to show
```js
db.products.find(
  { category: "sneakers" }, 
  { name: 1, price: 1, _id: 0 } //select fields
)
```


# 4. Sorting
```js
db.products.find().sort({ price: -1 }) //high to low
db.products.find().sort({ price: 1 }) //low to high
```
# 5. limit and skip
```js
db.products.find().limit(5) //pull first 5
db.products.find().skip(5).limit(5) //skip first 5 and pull next 5
```
# 6. Regex for finding text
```js
//find all product name that have "air" in side ex. "Air Jordan"
db.products.find({ name: /air/i })
```
# 7. Aggregation
### Aggregation Format
```js
db.products.aggregate([
	{ stage 1},
	{ stage 2},
	.	
	.
	{ stage n}
])
```
### $match -> filter
```js
//only category = sneakers
//can write like find()
db.products.aggregate([
  { $match: { category: "sneakers" } }
])
```

### $group -> group and count
```js
//basic syntax
db.collection.aggregate([
  {
    $group: {
      _id: <expression>,  // The field to group by
      <newField>: { <accumulator>: <expression> }
    }
  }
]);

db.products.aggregate([
  { $group:
	  {
		  _id: "$category", //group by catagory
		  avgPrice: { $avg: "$price" }, //find avg of price
		  prices: { $addToSet: "$airline.name"}, //create array of price
		  total: {$sum:"$price"}, //total price
		  number: {$sum:1} //number of item in group
	  }
  }
])
```
### $project -> select Fields
```js
// 1 -> select
// _id is always show at default set to 0 for disable
db.products.aggregate([
  { $project: { name: 1, price: 1, brand: 1, _id: 0 } }
])

//basic arimetric
db.orders.aggregate([
  {
    $project: {
      totalPrice: { $multiply: ["$price", "$quantity"] },
      discountedPrice: { $subtract: [{ $multiply: ["$price", "$quantity"] }, "$discount"] },
      finalAmount: { $add: [{ "$multiply": ["$price", "$quantity"] }, "$tax"] },
      profitMargin: { $divide: ["$profit", "$revenue"] }
    }
  }
])

//create TotalR fields for size of reviews
db.AirBnB.aggregate([ { $project: { TotalR: { $size: "$reviews" } } }])
```
### $sort
```js
//sort by price high to low
db.products.aggregate([
  { $sort: { price: -1 } }
])
```
### $limit & $skip -> for Pagination
```js
db.products.aggregate([
  { $skip: 5 },
  { $limit: 5 }
])
```
### $unwind -> separate each array values each documents
```js
//database
{
  "_id": 1,
  "customer": "John Doe",
  "items": ["Nike Air", "Adidas Boost", "Puma RS"]
}
```
```js
//query
db.orders.aggregate([
  { $unwind: "$items" }
])

//output
{ "_id": 1, "customer": "John Doe", "items": "Nike Air" }
{ "_id": 1, "customer": "John Doe", "items": "Adidas Boost" }
{ "_id": 1, "customer": "John Doe", "items": "Puma RS" }
```
### $lookup -> join with other collection
```js
//collection orders
{
  "_id": 1,
  "customer": "John Doe",
  "product_id": 101
}

//colleciton products
{
  "_id": 101,
  "name": "Nike Air",
  "price": 250
}

//join others with products
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "_id",
      as: "product_info"
    }
  }
])

//output data
{
  "_id": 1,
  "customer": "John Doe",
  "product_id": 101,
  "product_info": [
    {
      "_id": 101,
      "name": "Nike Air",
      "price": 250
    }
  ]
}
```
### using aggregation example
find most 3 customer total spent
```js
db.orders.aggregate([
  { $group: { _id: "$customer", total_spent: { $sum: "$total_price" } } },
  { $sort: { total_spent: -1 } },
  { $limit: 3 }
])
```


### `$expr` operator
-> `$expr` allows to use **aggregation expressions** inside `find()` query
### example
```js

//find document where array size is 50 or more
db.collection.find({
  $expr: { $gte: [{ $size: "$myArray" }, 50] }
})

//show all array "reviews" where size mte 50 and lte 99
db.AirBnB.find({ $expr: {
	$and: [
			{ $gte: [{ $size: "$reviews" }, 50] },
			{ $lte: [{ $size: "$reviews" }, 99] }
		] 
	} 
})

//all discountPrice less than price
db.collection.find({
  $expr: { $lt: ["$discountPrice", "$price"] }
})

```

