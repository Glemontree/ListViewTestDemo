# ListViewTestDemo
---


## 提升ListView的运行效率

。 在Adapter的getView()方法中，有一个参数convertView，这个参数用于将之前加载好的布局进行缓存，以便之后可以进行重用，这种方法可以解决在getView()方法中每次都重新加载布局的问题，当ListView快速滚动的时候这会成为性能的瓶颈。具体用法实例如下：

        public View getView(int position, View convertView, ViewGroup parent) {
            Fruit fruit = (Fruit) getItem(position);
            View view;
            if(convertView == null) {
                view = LayoutInflater.from(getContext()).inflate(resourceId, null);
            } else {
                view = convertView;
            }
            ImageView fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
            TextView fruitName = (TextView) view.findViewById(R.id.fruit_name);
            fruitImage.setResourceId(fruit.getImageId());
            fruitName.setText(fruit.getName());
            return view;
        }

**可以看到， 我们在getView()中对convertView进行了判断，如果为空，则使用LyaoutInflater去加载布局，否则，重用convertView.**

。 虽然使用convertView可以避免多次重新加载布局，但是每次getView()方法中还是会调用View的findViewById()方法获取控件的实例，这里可以采用ViewHolder来进行优化，具体代码示例如下：

    public class FruitAdapter extends ArrayAdapter<Fruit> {
        public View getView(int position, View convertView, ViewGroup parent) {
            Fruit fruit = (Fruit) getItem(position);
            View view;
            ViewHolder viewHolder;
            if(convertView == null) {
                viewHolder = new ViewHolder();
                view = LayoutInflater.from(getContext()).inflate(resourceId, null);
                viewHolder.fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
                viewHolder.fruitNmae = (TextView) view.findViewById(R.id.fruit_name);
                view.setTag(viewHolder);                                
            } else {
                view = convertView;
                viewHolder = view.getTag();                
            }
            viewHolder.fruitImage.setImageResource(fruit.getImageId());
            viewHolder.fruitName.setText(fruit.getName());
            return view;                      
        }
        class ViewHolder {
            ImageView fruitImage;
            TextView fruitName;    
        }
    }

**我们新增了一个内部类ViewHolder，用于对控件的实例进行缓存，当convertView为空的时候，创建了一个ViewHolder对象，并将控件的实例存放在ViewHolder中，然后调用View的setTag()方法将ViewHolder对象存储在View中，当convertView不为空的时候调用View的getTag()方法将ViewHolder重新取出。这样所有控件的实例都缓存在了ViewHolder里，就没有必要每次都通过findViewById()方法来获取控件实例了。**