package TCP;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.InetAddress;
//引入服务端的Serversocket包
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
/**
 * 传输协议
 * T：传输类型Type,文件，图片，字符串
 * L：长度length,记录文件占多少字节
 * D:二进制数据data
 * @author Administrator
 *
 */

public class Server {
	/**定义Serversocket变量
	 * 线程池：控制线程数量，线程重用
	 * 同步锁synchronized,可以修饰方法，可以变成synchronized块,多个线程看到的对象必须是同一个
	 * 双缓冲队列 offer pop，保证线程安全，效率高，就像两个管子
	 * 互斥所是synchronized锁的是同一个对象，且修饰代码或者方法不同是互斥锁
	 * 同步锁是synchronized锁的是同一个对象，且方法是一个是同步锁
	 */
	//保存所有用户输出流的集合
	private List<PrintWriter> allout;
	//运行在服务端的socket
	private ServerSocket server;
	//线程池，用于管理
	private ExecutorService threadpool;
	//构造方法中初始化服务端
	public Server() throws IOException{
		try{
			/**
			 * 创建serverSocket时需要只当端口号
			 */
			//System.out.println("初始化服务端");
	        server =new ServerSocket(8088);
	        //System.out.println("初始化完毕");
	        //创建线程池工具类Executors创建固定长度位50的线程池
	        threadpool=Executors.newFixedThreadPool(50);
	        //初始化所有客户端的集合,ArrayList利于遍历的  LinkList利于增删的
	        allout=new ArrayList<PrintWriter>();
	        
	        		
		}catch(Exception e){
			e.printStackTrace();
			throw e;
		}
	 }
	//将给定的字符流存入共享集合
	public synchronized void addout(PrintWriter pw){
		allout.add(pw);
	}
	
	//将给定的字符流从共享集合中删除
	public synchronized void removeout(PrintWriter pw){
			allout.remove(pw);
		}
	
	//遍历集合，群发消息
	public synchronized void sendmessage(String message){
				for(PrintWriter pw : allout){
					pw.println(message);
					
				}
			}
	
    public static void main(String[] args){
    	try{
        Server server=new Server();
        server.start();
    	}catch(Exception e){
    		e.printStackTrace();
    		System.out.println("服务器初始化失败");
    	}
    	
    }
    //服务端开始工作的
    public void start() {
    	try{
    		/**
    		 * accept方法用于监听8088端口，等待服务端连接
    		 * 该方法是一个阻塞方法，直到一个客户端连接，否者该方法一直阻塞
    		 * 若客户端连接了，回返回客户端的socket实例
    		 */
    		
    		/**
    		 * 当一个客户连接后，启动一个线程clientHandler，传入socket，启动一个服务端
    		 */
    		while(true){
    			System.out.println("等待客户端连接");
    		    Socket socket=server.accept();
    		    Runnable handler=new ClientHandler(socket);
    		    /**
    		     * Thread t=new Thread(handler);
    		     *   //启动线程
    		     * t.start();
    		     */
    		 
    		    //使用线程池分配空闲线程来处理当前连接的客户
    		    threadpool.execute(handler);
    		}
    			
    	
    		    
    	/**
    	 * //获取客户端的地址信息
    		InetAddress address=socket.getInetAddress();
    		//获取客户端的ip地址
    		String ha =address.getHostAddress();
    		int port=socket.getPort();
    		//System.out.println("客户端连上了");
    		//通过刚刚连上的客户端的socket获取输入流，来读取客户端发过来的信息
    		InputStream in=socket.getInputStream();
    		//获取字符串编码级
    		InputStreamReader isr=new InputStreamReader(in,"UTF-8");
    		//将字符串按照行的形式读取
    		BufferedReader br=new BufferedReader(isr);
    		//读会字符串
    		String message=null;
    		//判断读取到的字符串如果不是null的就直接输出
    		while((message=br.readLine())!=null){
    				System.out.println(ha+"-"+port+"客户端说:"+message);
    		}
    	 */
    	}catch(Exception e){
    		 e.printStackTrace();
    		 
    	}
    }
    //开启一个线程，重写run方法，使用线程的目的是服务端可以处理多个客户端
    class ClientHandler implements Runnable{
    	//当前线程处理的客户端的socket
    	private Socket socket;
    	//当前客户端的ip；
    	private String ip;
    	
    	public void run(){
    		PrintWriter pw=null;
           try{
        	   /**
        	    * 让服务端给客户端发送信息，通过socket获取输出流
        	    */
        	   //获取输出流
        	   OutputStream os=socket.getOutputStream();
        	   //指定输出字符集
        	   OutputStreamWriter ost=new OutputStreamWriter(os,"utf-8");
        	   //创建缓冲字符输出流  //println有自动行刷新的功能，添加一个参数true
        	   pw=new PrintWriter(ost,true);
        	   //将该客户端的输出流存入集合中，以便转发该客户端的消息
        	   addout(pw);
        	   
        	   
        	   
        	   //通过socket获取输入流
        	   InputStream in=socket.getInputStream();
        	   //socket输入流指定字符编码集
        	   InputStreamReader isr=new InputStreamReader(in,"UTF-8");
        	   //将字符以行的形式读取
        	   BufferedReader br=new BufferedReader(isr);
        	   //都出字符串
        	   String red=null;
        	   while((red=br.readLine())!=null){
        		   //System.out.println(red);     
        		   //pw.println(red);
        		   //读取到一条信息后转发给所有客户端
        		   sendmessage(red);
        	   } 
           }catch(Exception e){
        	   e.printStackTrace();
           }finally{
        	    //将该客户端的输出流从集合中删除
        	      removeout(pw);
        	      
        	      //输出当前在线人数
        	      System.out.println("当前在线人数："+allout.size());
	        	  try {
	        		  //运行完之后关掉流
					socket.close();
				} catch (Exception e) {
					e.printStackTrace();
				} 
	        	  
	        	  
	        	  System.out.println("该客户下线了");
           }
    	}
    	public ClientHandler(Socket socket){
    		//构造函数中初始化给定的当前启动的线程
    		this.socket=socket;
    		//获取客户端地址信息
    		InetAddress address=socket.getInetAddress();
    		//获取客户端的ip地址
    		ip =address.getHostAddress();
    		//获取客户端端口
    		int port=socket.getPort();
    		System.out.println(ip+":连上了");
    		sendmessage(ip+":上线了");
    	}
    }
}
