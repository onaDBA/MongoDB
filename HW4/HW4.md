#create database and collection
use DATA_POLICE_UK
db.createCollection("List_of_Forces")

#insert data
db.List_of_Forces.insert ([{"_id":8,"id":"bedfordshire8","name":"Bedfordshire Police8","opened":2023}])
db.List_of_Forces.insert ([{"_id":6,"id":"bedfordshire6","name":"Bedfordshire Police6"},{"_id":5,"id":"bedfordshire5","name":"Bedfordshire Police5", "opened":2023} ])
db.List_of_Forces.insert ([{"_id":7,"id":"bedfordshire7","name":["Bedfordshire Police7","Bedfordshire Police seven"]}])

#find data
db.List_of_Forces.find({"opened" : 2023})
db.List_of_Forces.find({$and :[{"opened" : 2023}, {"id": "bedfordshire8"}]})
db.List_of_Forces.find().limit(3)

#update data
db.List_of_Forces.update({"id" : "bedfordshire8"}, {$set: {"name": "Bedfordshire Police eight"}})
db.List_of_Forces.update({"_id" : 8}, {$set: {"rating": 7.5}})

#delete data
db.List_of_Forces.deleteMany({"_id": {$lt: 5}})

#drop documents, collection, db
db.List_of_Forces.deleteMany({})
db.List_of_Forces.drop()
db.dropDatabase()