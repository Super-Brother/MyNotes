```
@Test
	public void forEachTest() throws Exception{
		int max=100;
		outer:for(int i=0;i<max;i++){
			System.out.println("外层for循环,第("+(i+1)+")次循环开始++++++");
			if(i > 10){
				System.out.println("外层for循环,第("+(i+1)+")次循环,当前值i="+i);
				
				//开始内循环
				inner:for(int j=0;j<max;j++){
					System.out.println("内层for循环,第("+(j+1)+")次循环,当前值j="+j);
					if(j==5){
//						break inner;
						break outer;
					}
				}
			
				//结束循环
				if(i>30 && i%3 == 0){
					break outer;
				}
			}
			System.out.println("外层for循环,第("+(i+1)+")次循环结束++++++");
		}
```

