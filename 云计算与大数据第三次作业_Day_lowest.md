### 日最低气温的计算

- 将数据上传到hdfs
- 将编写好的java  tar包上传到hadoop编译运行
- 得最终结果
  - 日最低气温为

| 日期       | 最低气温 |
| ---------- | -------- |
| 2015.01.01 | -0.6     |
| 2015.01.02 | 1.3      |
| 2015.01.03 | 2.3      |
| 2015.01.04 | -1.3     |
| 2015.01.05 | -3.7     |
| 2015.01.06 | 2.9      |
| 2015.01.07 | -3.4     |
| 2015.01.08 | -7.9     |
| 2015.01.09 | 0.1      |
| 2015.01.10 | -2.0     |
| 2015.01.11 | 0.0      |
| 2015.01.12 | 1.4      |
| 2015.01.13 | -0.7     |
| 2015.01.14 | 0.9      |
| 2015.01.15 | 1.2      |
| 2015.01.16 | 3.5      |
| 2015.01.17 | 5.0      |
| 2015.01.18 | 7.6      |
| 2015.01.19 | 6.7      |
| 2015.01.20 | 9.5      |
| 2015.01.21 | 6.9      |
| 2015.01.22 | 3.5      |
| 2015.01.23 | 2.2      |
| 2015.01.24 | 1.4      |
| 2015.01.25 | 6.4      |
| 2015.01.26 | 7.2      |



- java代码如下

```java
import java.io.IOException;
import java.util.Iterator;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;

import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.conf.Configuration;

public class DayMin {
	
	public static class MinTemperatureMapper extends
			Mapper<LongWritable, Text, Text, Text> {
		
		@Override
		public void map(LongWritable arg0, Text Value, Context context)
				throws IOException, InterruptedException {

			
			String line = Value.toString();
			if (!(line.length() == 0)) {
				
				//date
				
				String date = line.substring(6, 14);
				
				float temp_Max = Float
						.parseFloat(line.substring(39, 45).trim());
				float temp_Min = Float
						.parseFloat(line.substring(47, 53).trim());
				if(Math.abs(temp_Min)!=9999){
					context.write(new Text(date+"的最低温度"),new Text(String.valueOf(temp_Min)));
				}
				
			}
		}

	}
	public static class MinTemperatureReducer extends
			Reducer<Text, Text, Text, Text> {

		public void reduce(Text Key, Iterator<Text> Values, Context context)
				throws IOException, InterruptedException {
			String temperature = Values.next().toString();
			context.write(Key, new Text(temperature));
		}

	}
	
	public static void main(String[] args) throws Exception {
                 
		Configuration conf = new Configuration();
		
				
		Job job = new Job(conf, "min weathertemputer");
		job.setJarByClass(MinTemperature.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		job.setMapperClass(MinTemperatureMapper.class);
		job.setReducerClass(MinTemperatureReducer.class);
		job.setInputFormatClass(TextInputFormat.class);
		job.setOutputFormatClass(TextOutputFormat.class);
		Path OutputPath = new Path(args[1]);
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		OutputPath.getFileSystem(conf).delete(OutputPath);
		System.exit(job.waitForCompletion(true) ? 0 : 1);

	}
}

```

