---
title: 用java写一个拼音模板生成器
name: java-pinyinchart
date: 2019-02-03
id: 0
tags: [java]
categories: []
abstract: ""
---
为了方便儿童认知拼音四线格，我用java写了一个简单的拼音字母模板

<!--more-->


为了方便儿童认知拼音四线格，我用java写了一个简单的拼音字母模板
<!--more-->

```java
import javax.imageio.ImageIO;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

//简单的拼音格制作
public class happy{
	public static String pinyinzimu;
	public static void main(String args[]){
		try {
			UIManager.setLookAndFeel("com.sun.java.swing.plaf.windows.WindowsLookAndFeel");
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (UnsupportedLookAndFeelException e) {
			e.printStackTrace();
		}

		JFrame frame=new JFrame("拼音模板");
		pinyinpanel panel=new pinyinpanel();
		frame.setBounds(100,100,1400,500);
		frame.setLayout(null);
		panel.setBounds(0,20,1330,210);
		frame.add(panel);
		final ImageIcon[] pinyin = {new ImageIcon("src/pinyin.png")};
		JLabel label=new JLabel();
		label.setText("abcd");
		label.setFont(new Font("微软雅黑",Font.PLAIN,120));
		//添加几个按钮用
		JButton b1=new JButton("添加字母");
		JButton b2=new JButton("保存图片");
		JButton b3=new JButton("清空内容");
		b1.setBounds(10,350,200,50);
		b2.setBounds(250,350,200,50);
		b3.setBounds(490,350,200,50);

		//添加监听
		b1.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(ActionEvent e) {
				JFrame frame1=new JFrame("输入字母");
				frame1.setBounds(200,300,500,200);
				frame1.setLayout(null);
				TextField textField=new TextField();
				textField.setEditable(true);
				textField.setText(label.getText());
				textField.setBounds(5,5,490,50);
				JButton add=new JButton("确定");
				add.setBounds(10,80,150,50);
				JButton del=new JButton("清空");
				del.setBounds(300,80,150,50);

				//添加监听
				add.addActionListener(new ActionListener() {
					@Override
					public void actionPerformed(ActionEvent e) {
						pinyinzimu =textField.getText();
						label.setText(pinyinzimu);
						frame1.setVisible(false);
					}
				});
				del.addActionListener(new ActionListener() {
					@Override
					public void actionPerformed(ActionEvent e) {
						textField.setText(null);
					}
				});


				frame1.add(textField);
				frame1.add(add);
				frame1.add(del);
				frame1.setVisible(true);
			}
		});

		b2.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(ActionEvent e) {
				//仅仅对panel进行截图
				BufferedImage map=new BufferedImage(panel.getWidth(),panel.getHeight(),BufferedImage.TYPE_INT_RGB);
				Graphics2D g=map.createGraphics();
				panel.printAll(g);
				JFileChooser jfc=new JFileChooser();
				jfc.showSaveDialog(null);
				String path=jfc.getSelectedFile().getPath();
					File f=new File(path+"."+"jpg");
					try{
						ImageIO.write(map,"jpg",f);
					}catch(IOException err){
						err.printStackTrace();
					}

			}
		});

		b3.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(ActionEvent e) {
				label.setText(null);
			}
		});

		frame.add(b1);
		frame.add(b2);
		frame.add(b3);

		panel.add(label);
		panel.setLayout(null);
		label.setBounds(10,-7,1400,200);
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		frame.setVisible(true);
	}
}

class pinyinpanel extends JPanel{
	//进行重绘
	public void paintComponent(Graphics g){
		super.paintComponent(g);
		ImageIcon imageIcon=new ImageIcon(this.getClass().getResource("pinyin.png"));
		g.drawImage(imageIcon.getImage(),0,0,1322,208,null);
		int step=200;//定义一个步进为100
		for (int i=0;i<6;i++) {
			step +=200;
			g.drawLine(10+step, 5, 10+step, 205);
		}

	}

}
```

最终效果图：

![](/images/java-pinyin1.webp)