محیط cli و کانکشن استرینگ :
mongosh mongodb://user:password@127.0.0.1:27017/db?authSource=admin


### backup
+ dump
  
      mongodump --host=192.168.13.32 --port=27018 --authenticationDatabase="admin" -u="root" -p="root" --db=local --collection=startup_log --gzip


  
