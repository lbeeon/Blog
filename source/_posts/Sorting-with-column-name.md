title: Sorting with column name
tags:
  - 'C#'
  - 擴充方法
  - 隨手筆記
date: 2015-09-19 01:27:31
---

## 隨手筆記

IQueryable和IEumerable相信是很多開發人員常用到的類別，並配合linq或是lambda來做資料處理，
不過每當資料要成現時，總免不要要排個序取個分頁，當欄位是固定的時候或許還能這樣寫

	var query  = data.OrderBy(p=>p.col1);

不過如果欄位很多樣的時侯就糗了，尤其當要排序的欄位是從client送上來的，多半會傳欄位名稱，無技可施的情況寫你可能會寫出這樣的程式碼

	switch(colname){
    	case col1:
        		query  = data.OrderBy(p=>p.col1);
        		break;
        case col2:
        		query  = data.OrderBy(p=>p.col2);
        		break;
        ......
        default:
        		query = data.OrderBy(p=> p.id);
    }
這時侯你就會希望linq本身提供的排序要是也能吃欄位名稱就好了...

當然我也是不例外，linq不吃字串排序使用起來實在是太不直覺了，還好皇天不負苦心人，終於讓我在[google](http://google.com),[stackoverflow](http://stackoverflow.com)的問題海中找到了，這篇主要就是要記錄這段短小精悍的程式碼，不過一開始提到的IQueryable和IEumerable的實作上是有一些差異，不囉嗦看程式碼
	
	public static class LinqExtensions
    {
        
        public static IOrderedQueryable<T> OrderBy<T>(this IQueryable<T> source, string propertyName)
        {
            return (IOrderedQueryable<T>)OrderBy((IQueryable)source, propertyName);
        }

        public static IQueryable OrderBy(this IQueryable source, string propertyName)
        {

            var x = Expression.Parameter(source.ElementType, "x");

            var selector = Expression.Lambda(Expression.PropertyOrField(x, propertyName), x);

            return source.Provider.CreateQuery(

                Expression.Call(typeof(Queryable),
                				"OrderBy",
                				new Type[] { source.ElementType, selector.Body.Type },
								source.Expression, selector));
        }

        public static IOrderedQueryable<T> OrderByDescending<T>(this IQueryable<T> source, string propertyName)
        {
            return (IOrderedQueryable<T>)OrderByDescending((IQueryable)source, propertyName);
        }

        public static IQueryable OrderByDescending(this IQueryable source, string propertyName)
        {

            var x = Expression.Parameter(source.ElementType, "x");

            var selector = Expression.Lambda(Expression.PropertyOrField(x, propertyName), x);

            return source.Provider.CreateQuery(

                Expression.Call(typeof(Queryable),
                				"OrderByDescending",
                                new Type[] { source.ElementType, selector.Body.Type },
								source.Expression, selector));
        }
    }

有了這個上面這段程式碼就可以直接寫成

	var query = data.OrderBy(colname);
    
網路上有很多種寫法，不過這種目前應該是最佳解，理由是，很多的OrderBy的寫法都是直接回傳IQueryable，而直接回傳IQueryable也沒有不好，不過你會無法接著使用ThenBy、ThenByDescending的方法。另外，當你將IQueryable實體化的時候，資料就變成了List，但是很多時候你要排序的結果都是List Array，而上面的方法就無法使用了，因此我就依樣畫葫替IEnumerable寫一段~~請小心服用~~，

	    public static class LinqExtensions
    {
	        public static IOrderedEnumerable<TSource> OrderBy<TSource>
            										(this IEnumerable<TSource> source, string propertyName)
        {

            PropertyInfo prop = typeof(TSource).GetProperty(propertyName);
            if (prop == null)
            {
                throw new Exception("No property '" + propertyName + "' in + " + typeof(TSource).Name + "'");
            }
            return source.OrderBy(x => prop.GetValue(x, null));
        }

        public static IOrderedEnumerable<TSource> OrderByDescending<TSource>
        											(this IEnumerable<TSource> source, string propertyName)
        {
            PropertyInfo prop = typeof(TSource).GetProperty(propertyName);
            if (prop == null)
            {
                throw new Exception("No property '" + propertyName + "' in + " + typeof(TSource).Name + "'");
            }
            return source.OrderByDescending(x => prop.GetValue(x, null));
        }
    }

藉由這兩個擴充方法，你就可以在linq中展現各種排序神技。[(GitHub)](https://github.com/lbeeon/LinqExtensions)
