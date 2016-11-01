router.get('/', function(req, res, next) {
    let querys=req.query;
    pg.connect(constr, function(err, client, done) {
        if (err) {
            console.log(err);
            res.send({status:err});
            return;
        }
        res.send({status:'ok'});
       client.end();
   });
});

when i request 10 times,the pg connect is no response and no err but could not connect...please help me 