# chat

Chat se basa un chat mediante un cliente servidor con el que se pueden conectar todas las personas que quieran y aparecerá su dirección de ip cuando envie algo.

##Aplicación del cliente
```java
  package servidor;
import java.net.*;
import java.io.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class Application1 {

  public Application1() throws IOException {
    Socket s = new Socket ("127.0.0.1",9090);
    Frame frame = new lomakyChat ("Chat " + "127.0.0.1" + ":" + "9090",
                    s.getInputStream (), s.getOutputStream ());    
    
//    Frame frame = new lomakyChat();
    //Center the window
    Dimension screenSize = Toolkit.getDefaultToolkit().getScreenSize();
    Dimension frameSize = frame.getSize();
    if (frameSize.height > screenSize.height) {
      frameSize.height = screenSize.height;
    }
    if (frameSize.width > screenSize.width) {
      frameSize.width = screenSize.width;
    }
    frame.setLocation((screenSize.width - frameSize.width)/2, (screenSize.height - frameSize.height)/2);
    frame.addWindowListener(new WindowAdapter() { public void windowClosing(WindowEvent e) { System.exit(0); } });
    frame.setVisible(true);
  }

  /**
   * main
   * @param args
   */
  public static void main(String[] args) throws IOException{
    try  {
      UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
    }
    catch (Exception e) {
      e.printStackTrace();
    }
    new Application1();
  }
}

```

##estructura del cliente
```java 
package servidor;

import java.net.*;
import java.io.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.*;

public class lomakyChat extends JFrame implements Runnable{
  BorderLayout borderLayout1 = new BorderLayout();
  JMenuBar menuBar1 = new JMenuBar();
  JMenu menuFile = new JMenu();
  JMenuItem menuFileExit = new JMenuItem();
  JMenu menuHelp = new JMenu();
  JMenuItem menuHelpAbout = new JMenuItem();
  JSplitPane jSplitPane1 = new JSplitPane();
  JSplitPane jSplitPane2 = new JSplitPane();
  JScrollPane jScrollPane1 = new JScrollPane();
  JScrollPane jScrollPane2 = new JScrollPane();
  JTextArea jTextArea1 = new JTextArea();
  JTextArea jTextArea2 = new JTextArea();
  JPanel jPanel1 = new JPanel();
  JLabel jLabel1 = new JLabel();
  JSplitPane jSplitPane3 = new JSplitPane();
  JScrollPane jScrollPane3 = new JScrollPane();
  JPanel jPanel2 = new JPanel();
  TextField input = new TextField();
  JTextArea output = new JTextArea();  
  JButton jButton1 = new JButton();
  DataInputStream i;
  DataOutputStream o;
  protected Thread listener;
  /**
   * Constructs a new instance.
   */
  public lomakyChat(String title, InputStream i, OutputStream o) {
    super("Lomaky Chat");
    this.i = new DataInputStream (new BufferedInputStream (i));
    this.o = new DataOutputStream (new BufferedOutputStream (o));
    try  {
      jbInit();
    }
    catch (Exception e) {
    }
    input.requestFocus ();
    listener = new Thread (this);
    listener.start ();     
  }

  /**
   * Initializes the state of this instance.
   */
  private void jbInit() throws Exception {
    this.getContentPane().setLayout(borderLayout1);
    this.setSize(new Dimension(495, 592));
    menuFile.setText("Salir...");
    menuFileExit.setText("Bye");
    jSplitPane1.setOrientation(JSplitPane.VERTICAL_SPLIT);
    jTextArea1.setText("");
    
    jTextArea2.setEditable(false);
    output.setEditable(false);
    jLabel1.setText("Chat");
    jLabel1.setFont(new Font("Dialog", 1, 40));
    jButton1.setText("Enviar!");
    menuFile.add(menuFileExit);
    menuBar1.add(menuFile);
    menuHelp.add(menuHelpAbout);
    menuBar1.add(menuHelp);
    this.setJMenuBar(menuBar1);
    this.getContentPane().add(jSplitPane1, BorderLayout.CENTER);
    jSplitPane1.add(jSplitPane2, JSplitPane.BOTTOM);
    jSplitPane2.add(jScrollPane1, JSplitPane.RIGHT);
    jScrollPane1.getViewport().add(jTextArea1, null);
    jScrollPane1.getViewport().add(output, null);
    jSplitPane2.add(jScrollPane2, JSplitPane.LEFT);
    jScrollPane2.getViewport().add(jTextArea2, null);
    jSplitPane1.add(jPanel1, JSplitPane.TOP);
    jPanel1.add(jLabel1, null);
    this.getContentPane().add(jSplitPane3, BorderLayout.SOUTH);
    jSplitPane3.add(jScrollPane3, JSplitPane.BOTTOM);
    jScrollPane3.getViewport().add(input, null);
    jSplitPane3.add(jPanel2, JSplitPane.TOP);
    jPanel2.add(jButton1, null);
//----------------------------------------------------------------------    
    menuFileExit.addActionListener(new ActionListener() {
      public void actionPerformed(ActionEvent e) {
        fileExit_ActionPerformed(e);
      }
    });  
//----------------------------------------------------------------------        
    jButton1.addActionListener(new ActionListener() {
      public void actionPerformed(ActionEvent e) {
      	try {
        	o.writeUTF (input.getText());
            o.flush (); 
            System.out.println("Enviando..");
      	} 
      	catch (IOException ex) {
        	ex.printStackTrace();
        	listener.stop ();
      	}
      	input.setText ("");
            
      }
    });    
//----------------------------------------------------------------------    
  }

  public void run () {
	Vector control = new Vector();
    String s;
    try {
      while (true) {
        String line = i.readUTF ();
		
        if (line.substring(0,2).equals("->")){
          	jTextArea2.setText("                  \n");
       		jTextArea2.append(line.substring(2));
       		
        }
        
        else{
	         if (line.substring(0,2).equals("<-")){
          	     jTextArea2.setText("                  \n");
          	     System.out.println("Salio "+line);
       		     jTextArea2.append(line.substring(2));
  	         }
  	         else
	             output.append(line + "\n");        	
        }
      }
    } catch (IOException ex) {
      ex.printStackTrace ();
    } finally {
      listener = null;
      input.hide ();
      validate ();
      try {
        o.close ();
      } catch (IOException ex) {
        ex.printStackTrace ();
      }
    }
  }

  public boolean handleEvent (Event e) {
    if ((e.target == input) && (e.id == Event.ACTION_EVENT)) {
      try {
        o.writeUTF ((String) e.arg);
            o.flush (); 
      } catch (IOException ex) {
        ex.printStackTrace();
        listener.stop ();
      }
      input.setText ("");
      return true;
    } else if ((e.target == this) && (e.id == Event.WINDOW_DESTROY)) {
      if (listener != null)
        listener.stop ();
      hide ();
      return true;
    }
    return super.handleEvent (e);
  }
  
//----------------------------------------------------------------------      
  void fileExit_ActionPerformed(ActionEvent e) {
    System.exit(0);
  }
}
```
##Servidor
```java 
  package servidor;
import java.net.*;
import java.io.*;
import java.util.*;


public class  Server  {
  public Server (int port) throws IOException {
	Vector x = new Vector();
    ServerSocket server = new ServerSocket (port);
    while (true) {
      Socket client = server.accept ();
      System.out.println ("Entro " + client.getInetAddress ());
      x.add(client.getInetAddress ());
      ChatHandler c = new ChatHandler (client,x);
      c.start ();
    }
  }
  public static void main (String args[]) throws IOException {
    new Server (9090);
  }
} 


```

##Thread del servidor
```java 
  package servidor;

import java.net.*;
import java.io.*;
import java.util.*;

public class  ChatHandler  extends Thread {
  protected Socket s;
  protected DataInputStream i;
  protected DataOutputStream o;
  Vector y = new Vector();

  public ChatHandler (Socket s, Vector x) throws IOException {
    this.s = s;
    i = new DataInputStream (new BufferedInputStream (s.getInputStream ()));
    o = new DataOutputStream (new BufferedOutputStream (s.getOutputStream ()));
    y = x;
  }

  protected static Vector handlers = new Vector ();

  
  public void run () {
    String name = s.getInetAddress ().toString ();
    String Listado="";
    try {
    	
	  for (int i=0; i < y.size() ; i++)
	  	Listado=Listado+y.get(i)+"\n";
	  
      
      
      handlers.addElement (this);
      broadcast ("->" + Listado);
      while (true) {
        String msg = i.readUTF ();
        broadcast (name + " - " + msg);
      }
    } catch (IOException ex) {
      ex.printStackTrace ();
    } finally {
      handlers.removeElement (this);
      y.remove(name);
      Listado="";
	  for (int i=0; i < y.size() ; i++)
	  	Listado=Listado+y.get(i)+"\n";
      broadcast ("<-" + Listado);
      try {
        s.close ();
      } catch (IOException ex) {
        ex.printStackTrace();
      }
    }
  }

  protected static void broadcast (String message) {
    synchronized (handlers) {
      Enumeration e = handlers.elements ();
      while (e.hasMoreElements ()) {
        ChatHandler c = (ChatHandler) e.nextElement ();
        try {
          synchronized (c.o) {
            c.o.writeUTF (message);
          }
          c.o.flush ();
        } catch (IOException ex) {
          c.stop ();
        }
      }
    }
  }
} 

```
