# org.springframework.data.repository 配合事物@Transactional的一个坑

## 在有事物的情况下

### controller层

当我根据planId查询Plan的某一条数据时（mysql数据库），如果此时传参数`bindCode`去修改`CustomerId`时，数据库里面这条数据的这个字段也会同时修改，见下面的sql查询结果

```java
@ApiOperation("error")
@PostMapping("error")
@Transactional(rollbackFor = Exception.class)
public Object error(@RequestBody JSONObject jsonObject) {
    Optional<Plan> one = myPlanRepository.findById(jsonObject.getLong("planId"));
    one.ifPresent(v->{
        v.setCustomerId(jsonObject.getString("bindCode"));
    });
    return one.get();
}
```

### Repository层

我的myPlanRepository继承了CrudRepository

![image-20211229102351864](https://gitee.com/laoyouji1018/images/raw/master/img/202112291023631.png)

### 测试

原始的数据库数据如下

![image-20211229103102478](https://gitee.com/laoyouji1018/images/raw/master/img/202112291031513.png)

当我传如下参数的时候

![image-20211229103149492](https://gitee.com/laoyouji1018/images/raw/master/img/202112291031519.png)

此时的数据库该字段的数据变了！！！

![image-20211229103258309](https://gitee.com/laoyouji1018/images/raw/master/img/202112291032337.png)

## 在无事物的情况下

### controller层

```java
@ApiOperation("error")
@PostMapping("error")
public Object error(@RequestBody JSONObject jsonObject) {
    Optional<Plan> one = myPlanRepository.findById(jsonObject.getLong("planId"));
    one.ifPresent(v->{
        v.setCustomerId(jsonObject.getString("bindCode"));
    });
    return one.get();
}
```

### 测试

尝试去修改`CustomerId`的值，改为333

![image-20211229103458988](https://gitee.com/laoyouji1018/images/raw/master/img/202112291034023.png)

此时数据库该字段并没有变化！！！是不是很懵逼！！！

![image-20211229103548264](https://gitee.com/laoyouji1018/images/raw/master/img/202112291035297.png)

## save方法和查询方法同时存在时

### controller层

方法如下

```java
@ApiOperation("error")
@PostMapping("error")
public Object error(@RequestBody JSONObject jsonObject) {
    Optional<Plan> one = myPlanRepository.findById(jsonObject.getLong("planId"));
    myPlanRepository.save(one.get());
    return one.get();
}
```

`save`调用的是`CrudRepository`下的`save`

![image-20211229104049473](https://gitee.com/laoyouji1018/images/raw/master/img/202112291040509.png)

### 测试

参数如下

![image-20211229104418528](https://gitee.com/laoyouji1018/images/raw/master/img/202112291044559.png)

我的idea控制台打印的sql日志

> -2021-12-29 10:45:30.144 - INFO 1613 [512dc7b3-2ae1-4168-9004-bb27b553b989 session=a08bd73c-5295-4e17-827d-7be84e54d8a0] {magenta} --- [nio-4000-exec-1] ybplan.aspect.LogAspect                  : staff : null method : SqlController.error[[{"planId":1806}]] key : null before
> -Hibernate: 
>     select
>         plan0_.id as id1_35_0_,
>         plan0_.create_date_time as create_d2_35_0_,
>         plan0_.del as del3_35_0_,
>         plan0_.modify_date_time as modify_d4_35_0_,
>         plan0_.cover_path as cover_pa5_35_0_,
>         plan0_.customer_id as customer6_35_0_,
>         plan0_.default_policy_year as default_7_35_0_,
>         plan0_.emp_name as emp_name8_35_0_,
>         plan0_.emp_phone as emp_phon9_35_0_,
>         plan0_.empno as empno10_35_0_,
>         plan0_.get_way as get_way11_35_0_,
>         plan0_.has_bonus_level as has_bon12_35_0_,
>         plan0_.has_get_setting as has_get13_35_0_,
>         plan0_.has_share_setting as has_sha14_35_0_,
>         plan0_.holder_age as holder_15_35_0_,
>         plan0_.holder_birthdate as holder_16_35_0_,
>         plan0_.holder_insured_relation as holder_17_35_0_,
>         plan0_.holder_name as holder_18_35_0_,
>         plan0_.holder_sex as holder_19_35_0_,
>         plan0_.iflytek_url as iflytek20_35_0_,
>         plan0_.img_case as img_cas21_35_0_,
>         plan0_.img_url as img_url22_35_0_,
>         plan0_.insure_qr_code as insure_23_35_0_,
>         plan0_.insured_age as insured24_35_0_,
>         plan0_.insured_birthdate as insured25_35_0_,
>         plan0_.insured_name as insured26_35_0_,
>         plan0_.insured_sex as insured27_35_0_,
>         plan0_.plan_code as plan_co28_35_0_,
>         plan0_.plan_status as plan_st29_35_0_,
>         plan0_.prod_code as prod_co30_35_0_,
>         plan0_.prod_name as prod_na31_35_0_,
>         plan0_.prod_num as prod_nu32_35_0_,
>         plan0_.prod_year_num as prod_ye33_35_0_,
>         plan0_.product_id as product34_35_0_,
>         plan0_.share_id as share_i35_35_0_,
>         plan0_.total_amount as total_a36_35_0_,
>         plan0_.total_premium as total_p37_35_0_ 
>     from
>         plan plan0_ 
>     where
>         plan0_.id=? 
>         and (
>             plan0_.del = 0
>         )
> -2021-12-29 10:45:30.456 - INFO 1613 [512dc7b3-2ae1-4168-9004-bb27b553b989 session=a08bd73c-5295-4e17-827d-7be84e54d8a0] {magenta} --- [nio-4000-exec-1] ybplan.aspect.LogAspect                  : staff : null method :
> -

你会发现只执行了查询的方法，并没有执行更新的方法，猜测可能的原因是我们没有修改查询出`Plan`的属性值，由此我们进行如下实验，将查询出`plan`的`customerId`的值修改了

```java
@ApiOperation("error")
    @PostMapping("error")
    public Object error(@RequestBody JSONObject jsonObject) {
        Optional<Plan> one = myPlanRepository.findById(jsonObject.getLong("planId"));
        one.ifPresent(v->{
            v.setCustomerId(jsonObject.getString("bindCode"));
        });
        myPlanRepository.save(one.get());
        return one.get();
    }
```

继续测试，参数如下

![image-20211229105159185](https://gitee.com/laoyouji1018/images/raw/master/img/202112291051228.png)

查看idea控制台日志

> -Hibernate: 
>     select
>         plan0_.id as id1_35_0_,
>         plan0_.create_date_time as create_d2_35_0_,
>         plan0_.del as del3_35_0_,
>         plan0_.modify_date_time as modify_d4_35_0_,
>         plan0_.cover_path as cover_pa5_35_0_,
>         plan0_.customer_id as customer6_35_0_,
>         plan0_.default_policy_year as default_7_35_0_,
>         plan0_.emp_name as emp_name8_35_0_,
>         plan0_.emp_phone as emp_phon9_35_0_,
>         plan0_.empno as empno10_35_0_,
>         plan0_.get_way as get_way11_35_0_,
>         plan0_.has_bonus_level as has_bon12_35_0_,
>         plan0_.has_get_setting as has_get13_35_0_,
>         plan0_.has_share_setting as has_sha14_35_0_,
>         plan0_.holder_age as holder_15_35_0_,
>         plan0_.holder_birthdate as holder_16_35_0_,
>         plan0_.holder_insured_relation as holder_17_35_0_,
>         plan0_.holder_name as holder_18_35_0_,
>         plan0_.holder_sex as holder_19_35_0_,
>         plan0_.iflytek_url as iflytek20_35_0_,
>         plan0_.img_case as img_cas21_35_0_,
>         plan0_.img_url as img_url22_35_0_,
>         plan0_.insure_qr_code as insure_23_35_0_,
>         plan0_.insured_age as insured24_35_0_,
>         plan0_.insured_birthdate as insured25_35_0_,
>         plan0_.insured_name as insured26_35_0_,
>         plan0_.insured_sex as insured27_35_0_,
>         plan0_.plan_code as plan_co28_35_0_,
>         plan0_.plan_status as plan_st29_35_0_,
>         plan0_.prod_code as prod_co30_35_0_,
>         plan0_.prod_name as prod_na31_35_0_,
>         plan0_.prod_num as prod_nu32_35_0_,
>         plan0_.prod_year_num as prod_ye33_35_0_,
>         plan0_.product_id as product34_35_0_,
>         plan0_.share_id as share_i35_35_0_,
>         plan0_.total_amount as total_a36_35_0_,
>         plan0_.total_premium as total_p37_35_0_ 
>     from
>         plan plan0_ 
>     where
>         plan0_.id=? 
>         and (
>             plan0_.del = 0
>         )
> Hibernate: 
>     update
>         plan 
>     set
>         create_date_time=?,
>         del=?,
>         modify_date_time=?,
>         cover_path=?,
>         customer_id=?,
>         default_policy_year=?,
>         emp_name=?,
>         emp_phone=?,
>         empno=?,
>         get_way=?,
>         has_bonus_level=?,
>         has_get_setting=?,
>         has_share_setting=?,
>         holder_age=?,
>         holder_birthdate=?,
>         holder_insured_relation=?,
>         holder_name=?,
>         holder_sex=?,
>         iflytek_url=?,
>         img_case=?,
>         img_url=?,
>         insure_qr_code=?,
>         insured_age=?,
>         insured_birthdate=?,
>         insured_name=?,
>         insured_sex=?,
>         plan_code=?,
>         plan_status=?,
>         prod_code=?,
>         prod_name=?,
>         prod_num=?,
>         prod_year_num=?,
>         product_id=?,
>         share_id=?,
>         total_amount=?,
>         total_premium=? 
>     where
>         id=?

会发现此时执行了更新方法，由此可以得出结论，Hibernate对查询出的实体类进行监控，如果实体类变更了属性值，则会执行相应的更新，如果没有变化，则不会执行相应的更新

备注：经过测试save方法和查询方法同时存在的情况下，有事物无事物的结果都是一样的

by：三分钟热度