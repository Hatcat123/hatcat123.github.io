环境描述：




如果你的表比较小的话，可以对每个id单独查询，然后组成一个list：

[Shoe.query.filter_by(id=id).one() for id in my_list_of_ids]
1
如果你的表很大，上述查询方法会很慢。此时建议使用in条件的查询

shoes = Shoe.query.filter(Shoe.id.in_(my_list_of_ids)).all()


