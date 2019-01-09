

# Git

## git warning: LF will be replaced by CRLF in 
windows中的换行符为 CRLF, 而在Linux下的换行符为LF,所以在执行add . 时出现提示,

git warning: LF will be replaced by CRLF in 解决办法:  
$ rm -rf .git // 删除.git   
$ git config --global core.autocrlf false //禁用自动转换  

然后重新执行:  
$ git init  
$ git add   


# 野火仿真器  
##  swd/jtag communication failure  
- Error:Flash download failed - Target DLL has been cancelled  
原因：没有给电路板供电

# HT1621液晶显示芯片
我在给液晶全写1或0时发现，错误。
![image](https://note.youdao.com/yws/api/personal/file/908A7F6CEDC649DA96198CE508ACB081?method=download&shareKey=a63647f7d846709432c49fedb89c3201)  
按照时序写数据给HT1621时出现错误的部分代码CODE：  
```
void display_clr()
{
	unsigned char addr = 0;
	GPIO_ResetBits(GPIOA,HT1621_CS);
	Display_Write_Mode(1);
	
	for(addr = 0;addr < 32 ;addr ++)
	{
        Display_Write_address(addr);
		Display_Write_data_4bit(0);
	}
	
	GPIO_SetBits(GPIOA,HT1621_CS);
}
```
而按照连续写数据的时序的程序则执行成功，而这段代码按照我的理解地址不需要在进行写入，只需写数据即可；可是，当我把for循环去掉时，发现液晶显示还是错误。
  
![image](https://note.youdao.com/yws/api/personal/file/3403FE8777EE4BED84365FC79E3C72E8?method=download&shareKey=b1467a6267639b373fcbe4c90e017dd9)  
部分code如下：  
```
void display_clr()
{
	unsigned char addr = 0;
	GPIO_ResetBits(GPIOA,HT1621_CS);
	Display_Write_Mode(1);
	
	Display_Write_address(addr);
	for(addr = 0;addr < 32 ;addr ++)
	{

		Display_Write_data_4bit(0);
	}
	
	GPIO_SetBits(GPIOA,HT1621_CS);
}
```  
- 尝试一
将mode和address子程序的调用都放入for循环中，并且在循环最后加入延时120*2.1 us但是失败了。

- 尝试二（解决）  
仔细查看第一幅时序图，发现在写玩4bit数据后，CS被拉高一段时间后再拉低，再程序中添加cs= 1,delay,cs=0,液晶屏果然清屏了。
```
void Display_Write_data_4bit(unsigned char Dbyte)
{
	int i = 0;
	
	for(i = 0; i < 4; i ++ )
	{
		GPIO_ResetBits(GPIOA,HT1621_WR);
		Delay (60);
		if(	Dbyte >> (3 - i) & 0x01)	
		{
			GPIO_SetBits(GPIOA,HT1621_DAT);
		}	
		else 
			GPIO_ResetBits(GPIOA,HT1621_DAT);
		Delay (60);
		GPIO_SetBits(GPIOA,HT1621_WR);
		Delay(60);
		
	}
}
```

# C语言数组
每个数组的最后一个都是0，（还没有理解）。
