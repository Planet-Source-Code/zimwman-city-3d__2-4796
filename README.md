<div align="center">

## City 3D


</div>

### Description

3D City
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[zimwman](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/zimwman.md)
**Level**          |Beginner
**User Rating**    |5.0 (15 globes from 3 users)
**Compatibility**  |Java \(JDK 1\.1\), Java \(JDK 1\.2\), Java \(JDK 1\.3\), Java \(JDK 1\.4\), Java \(JDK 1\.5\)
**Category**       |[Applet](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/applet__2-81.md)
**World**          |[Java](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/java.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/zimwman-city-3d__2-4796/archive/master.zip)





### Source Code

```
/**
 * @(#)Cidade3D.java
 *
 * Sample Applet application
 *
 * @author Victor Santos
 * @version 3.00 05/04/21
 */
import java.awt.*;
import java.applet.*;
import java.awt.event.*;
import java.util.*;
class Rua {
	public int px, py;
	public int ye, yd;
	Color ce, cd;
	public Rua n,s,e,w,nx;
	public Rua(Point pon) {
		px=pon.x; py=pon.y;
		n=s=e=w=nx=null;
	}
	public Rua(int X, int Y) {
		this(new Point(X,Y));
	}
	public Rua() {
		this(new Point(0,0));
	}
}
class Carro {
	public int dx,dy;
	public int x, y, lx, ly, dir, cnt, sta;
	public Color cor;
	public Rua r;
	public Carro() {
		x=y=lx=ly=15;
	}
	public void clone(Carro t) {
		t.dx=dx;
		t.dy=dy;
		t.x=x;
		t.y=y;
		t.lx=lx;
		t.ly=ly;
		t.dir=dir;
		t.cnt=cnt;
		t.sta=sta;
		t.cor=cor;
		t.r=r;
	}
}
class pix3d {
	public int x,y,z;
	public pix3d(int X, int Y, int Z) {
		x=X; y=Y; z=Z;
	}
	public pix3d() {
		x=0; y=0; z=0;
	}
	public void clone(pix3d t) {
		t.x=x; t.y=y; t.z=z;
	}
}
class roomvar {
	int base, offst, e, d, le, ld, ne, nd, nt;
	Color ce, cd, nce, ncd, nct;
	public roomvar(int BASE, int OFFST, int E, int D, int LE, int LD, int NE,
			int ND, int NT, Color CE, Color CD, Color NCE, Color NCD, Color NCT) {
		base=BASE; offst=OFFST; e=E; d=D; le=LE; ld=LD; ce=CE; cd=CD;
		ne=NE; nd=ND; nt=NT; nce=NCE; ncd=NCD; nct=NCT;
	}
}
public class Cidade3D extends Applet implements Runnable, KeyListener {
  public static final int N = (int) 1 ;
	public static final int S = (int) 2 ;
	public static final int E = (int) 3 ;
	public static final int W = (int) 4 ;
	Rua raiz;
	int Max=500;
	Carro c[] = new Carro[Max], lc= new Carro();
	int Nqx=55, Nqz=55, Bdiv=6, Lbx=6, Lbz=6, Dim=10, Nlin=12, Pasz=60, PRF=1000;
	int Lx=300, Ly=100, Lenz=600, Lx_2=Lx/2, Lx_4=Lx/4, Lx_24=Lx_2+Lx_4;
	int width, height, width2, height2, remCt=0, remY;
	Color skyc=new Color(0.2f, 0.4f, 0.9f), remC;
	boolean refresh=true, vista2d=false;
	Image backbuffer;
	Graphics backg;
	Vector listOfroom;
	Thread artist=null;
	public void init() {
		resize(570,570);
		width = getSize().width;
		height = getSize().height;
		width2=width/2;
		height2=height/2;
		raiz=null;
		mapaInit();
		carrosInit();
		c[0].clone(lc);
		setBackground( Color.blue );
		backbuffer = createImage( width, height );
		backg = backbuffer.getGraphics();
		putmapa();
		listOfroom = new Vector();
		addKeyListener( this );
	}
	public void paint(Graphics g) {
		if(refresh && vista2d) {
			g.drawImage( backbuffer, 0, 0, this );
			refresh=false;
		}
		carrosMove();
		if(vista2d) {
			g.setColor(Color.black);
			g.fillRect(c[0].lx-2, c[0].ly-1, 3, 3);
			g.fillRect(c[1].lx-2, c[0].ly-1, 3, 3);
			g.setColor(Color.white);
			g.fillRect(c[0].x-2, c[0].y-1, 3, 3);
			g.fillRect(c[1].x-2, c[0].y-1, 3, 3);
			for(int i=2;i<Max;i++) {
				g.setColor(Color.black);
				g.fillRect(c[i].lx, c[i].ly, 2, 2);
				g.setColor(c[i].cor);
				g.fillRect(c[i].x, c[i].y, 2, 2);
			}
			showStatus( "2D View, Press ' V ' for 3D View " );
		}
		else {
			backg.setColor(skyc);
			backg.fillRect(0,0,width,height2);
			backg.setColor(Color.black);
			backg.fillRect(0,height2,width,height);
			Set3DView();
			g.drawImage( backbuffer, 0, 0, this );
			showStatus( "3D View at Pos: ("+c[0].r.px+" , "+c[0].r.py+") --> Press ' V ' for 2D View " );
		}
	}
	public void update(Graphics g) {
 		paint(g);
	}
	public void start() {
		if (artist == null) {
  		 	artist = new Thread(this);
  		artist.start();
 		}
		refresh=true;
	}
	public void stop() {
  	artist = null;
	}
	public void run() {
 		while (artist != null) {
  	try {Thread.sleep(10);} catch (InterruptedException e){}
  	repaint();
 		}
 		artist = null;
 		if(!vista2d) {
 		}
	}
	public void keyPressed( KeyEvent e ) {
		if(e.getKeyChar()=='v') {
			if(vista2d) {
				backg.setColor(Color.black);
	 			backg.fillRect(0,0,width,height);
				vista2d=false;
			}
			else {
				backg.clearRect(0,0,width,height);
				putmapa();
				refresh=true;
				vista2d=true;
			}
		}
		e.consume();
	}
	public void keyReleased( KeyEvent e ) { }
	public void keyTyped( KeyEvent e ) { }
	public void mapaInit() {
	int mx,mz, u;
	Rua t;
	for(mz=0;mz<Nqz;mz++)
		for(mx=0;mx<Nqx;mx++)
			if((mz%Bdiv)==0 || (mx%Bdiv)==0) {
				t=new Rua(mx+1,mz+1);
				t.ye=1+(int)(250*Math.random());
				t.yd=1+(int)(250*Math.random());
				t.ce=new Color( (float)Math.random(), (float)Math.random(), (float)Math.random() );
				t.cd=new Color( (float)Math.random(), (float)Math.random(), (float)Math.random() );
				if((mz%Bdiv)==0 && mx!=0) {     // L
					t.w=raiz;
					t.w.e=t;
				}
				if((mx%Bdiv)==0 && mz!=0) {
					t.n=findrua(mx+1,mz);	 		// ^
					t.n.s=t;
				}
				t.nx=raiz;
				raiz=t;
			}
	// Esquinas
	for(Rua r=raiz;r!=null;r=r.nx) {
		if(r.n==null) {
			if(r.w!=null)
				if(r.w.n!=null) { r.w.n.cd=r.ce; r.w.n.yd=r.ye; }
			if(r.e!=null)
				if(r.e.n!=null) { r.e.n.ce=r.ce; r.e.n.ye=r.ye; }
		}
		if(r.s==null) {
			if(r.w!=null)
				if(r.w.s!=null) { r.w.s.cd=r.cd; r.w.s.yd=r.yd; }
			if(r.e!=null)
				if(r.e.s!=null) { r.e.s.ce=r.cd; r.e.s.ye=r.yd; }
		}
		if(r.e==null) {
			if(r.n!=null)
				if(r.n.e!=null) { r.n.e.cd=r.cd; r.n.e.yd=r.yd; }
			if(r.s!=null)
				if(r.s.e!=null) { r.s.e.ce=r.cd; r.s.e.ye=r.yd; }
		}
		if(r.w==null) {
			if(r.n!=null)
				if(r.n.w!=null) { r.n.w.cd=r.ce; r.n.w.yd=r.ye; }
			if(r.s!=null)
				if(r.s.w!=null) { r.s.w.ce=r.ce; r.s.w.ye=r.ye; }
		}
	}
	}
public void putmapa() {
		for(Rua r=raiz;r!=null;r=r.nx) {
			backg.setColor( Color.black );
			backg.fillRect(r.px*Dim, r.py*Dim, Dim, Dim);
			if(r.n==null) {
				backg.setColor( r.ce );
				backg.fillRect(r.px*Dim,r.py*Dim-2,Dim, 2);
			}
			if(r.s==null) {
				backg.setColor( r.cd );
				backg.fillRect(r.px*Dim,r.py*Dim+Dim,Dim, 2);
			}
			if(r.e==null) {
				backg.setColor( r.cd );
				backg.fillRect(r.px*Dim+Dim,r.py*Dim,2, Dim);
			}
			if(r.w==null) {
				backg.setColor( r.ce );
				backg.fillRect(r.px*Dim-2,r.py*Dim,2, Dim);
			}
		}
}
	public Rua findrua(int fx,int fz) {
		for(Rua f=raiz;f!=null;f=f.nx)
			if(f.px==fx && f.py==fz) return f;
		return null;
	}
	public void carrosInit() {
		for(int n=0;n<Max;n++) {
			c[n]=new Carro();
			novo(c[n]);
			c[n].r=novoR();
			do c[n].dir=(int)(Math.random()*4)+1;
			while((c[n].dir==N && c[n].r.n==null) || (c[n].dir==S && c[n].r.s==null) ||
					(c[n].dir==E && c[n].r.e==null) || (c[n].dir==W && c[n].r.w==null));
			c[n].cnt=2;
			c[n].sta=0;
			c[n].cor=new Color( Color.HSBtoRGB((float)Math.random(),1,1) );
		}
	}
	public void novo(Carro t) {
		Point pon;
		do {
			pon=new Point((int)(Math.random()*Nqx)+1,(int)(Math.random()*Nqz)+1);
		} while(findrua(pon.x,pon.y)==null);
		t.dx=pon.x; t.dy=pon.y;
	}
	public Rua novoR() {
		Point pon=new Point();
		Rua rua;
		do {
			pon.x=(int)(Math.random()*Nqx)+1;
			pon.y=(int)(Math.random()*Nqz)+1;
			rua=findrua(pon.x,pon.y);
		} while(rua==null);
		return rua;
	}
	public void carrosMove() {
		c[1].clone(c[0]);
		for(int n=1;n<Max;n++) {
			if(c[n].cnt<Dim) c[n].cnt++;
			else {
				c[n].cnt=1;
				novarua(c[n]);
			}
			carrosSetPos(c[n]);
			if(c[n].r.px==c[n].dx && c[n].r.py==c[n].dy)
				novo(c[n]);
		}
	}
	public void novarua(Carro p) {
	Carro tc=new Carro();
	switch(p.dir) {
		case E : if(p.r.n==null && p.r.s==null) p.r=p.r.e;
			 else p.r=decide(p);
			 p.clone(tc);
			 if(tc.r.n!=null || tc.r.s!=null) {
				decide(tc);
				p.sta=tc.dir;
			 }
			 break;
		case W : if(p.r.n==null && p.r.s==null) p.r=p.r.w;
			 else p.r=decide(p);
			 p.clone(tc);
			 if(tc.r.n!=null || tc.r.s!=null) {
				decide(tc);
				p.sta=tc.dir;
			 }
			 break;
		case N : if(p.r.e==null && p.r.w==null) p.r=p.r.n;
			 else p.r=decide(p);
			 p.clone(tc);
			 if(tc.r.e!=null || tc.r.w!=null) {
				decide(tc);
				p.sta=tc.dir;
			 }
			 break;
		case S : if(p.r.e==null && p.r.w==null) p.r=p.r.s;
			 else p.r=decide(p);
			 p.clone(tc);
			 if(tc.r.e!=null || tc.r.w!=null) {
				decide(tc);
				p.sta=tc.dir;
			 }
			 break;
	}
	}
	public void carrosSetPos(Carro p) {
	int bx=p.r.px*Dim, by=p.r.py*Dim;
	int vx=0,vy=0;
	if(p.cnt>=Dim && p.sta==0) return;
	switch(p.dir) {
		case N : vx=7; vy=Dim-p.cnt;
			 switch(p.sta) {
				case E : vx+=p.cnt;
						if(p.cnt==2) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
				case W : vx-=p.cnt;
					 if(p.cnt==7) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
			 }
			 break;
		case S : vx=2; vy=p.cnt;
			 switch(p.sta) {
				case E : vx+=p.cnt;
					 if(p.cnt==7) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
				case W : vx-=p.cnt;
						if(p.cnt==2) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
			 }
			 break;
		case E : vx=p.cnt; vy=7;
			 switch(p.sta) {
				case N : vy-=p.cnt;
					 if(p.cnt==7) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
				case S : vy+=p.cnt;
						if(p.cnt==2) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
			 }
			 break;
		case W : vx=Dim-p.cnt; vy=2;
			 switch(p.sta) {
				case N : vy-=p.cnt;
						if(p.cnt==2) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
				case S : vy+=p.cnt;
					 if(p.cnt==7) {
						p.cnt=Dim;
						p.sta=0;
					 }
					 break;
			 }
			 break;
	}
	p.lx=p.x;
	p.x=bx+vx;
	p.ly=p.y;
	p.y=by+vy;
	}
	public Rua decide(Carro p) {
	int dix=p.r.px-p.dx, diy=p.r.py-p.dy, dax=Math.abs(dix), day=Math.abs(diy);
	switch(p.dir) {
		case N : if(diy>=Lbz || (diy>0 && day>=dax)) return p.r.n;
			 if(p.r.e!=null && dix<=0) {
				p.dir=E;
				return p.r.e;
			 }
			 if(p.r.w!=null && dix>=0) {
				p.dir=W;
				return p.r.w;
			 }
			 break;
		case S : if(diy<=-Lbz || (diy<0 && day>=dax)) return p.r.s;
			 if(p.r.e!=null && dix<=0) {
				p.dir=E;
				return p.r.e;
			 }
			 if(p.r.w!=null && dix>=0) {
				p.dir=W;
				return p.r.w;
			 }
			 break;
		case E : if(dix<=-Lbx || (dix<0 && dax>=day)) return p.r.e;
			 if(p.r.s!=null && diy<=0) {
				p.dir=S;
				return p.r.s;
			 }
			 if(p.r.n!=null && diy>=0) {
				p.dir=N;
				return p.r.n;
			 }
			 break;
		case W : if(dix>=Lbx || (dix>0 && dax>=day)) return p.r.w;
			 if(p.r.s!=null && diy<=0) {
				p.dir=S;
				return p.r.s;
			 }
			 if(p.r.n!=null && diy>=0) {
				p.dir=N;
				return p.r.n;
			 }
			 break;
	}
	return null;
	}
	public void Set3DView() {
	int d, e, ld, le, nd, ne, nt;
	Color ce, cd, nce=Color.red, ncd=Color.red, nct=Color.red;
	int[] polyX=new int[4];
	int[] polyY=new int[4];
	Rua r=null;
	if(lc.cnt!=c[0].cnt) {
		d=0; e=0; nd=0; ne=0; nt=0;
		r=c[0].r;
		ce=r.ce; cd=r.cd;
		for(int n=0;n<Nlin && r!=null; n++) {
			ld=d; le=e;
			switch(c[0].dir) {
				case N : if(r.n==null) { nt=r.ye; nct=r.ce; }
						 else nt=0;
						 if(r.e==null) { d=r.yd; cd=r.cd; nd=0; }
						 else {
						 	 d=0;
						 	 if(nt==0) { nd=r.e.e.ye; ncd=r.e.e.ce; }
						 	 else { nd=r.e.ye; ncd=r.e.ce; }
						 }
						 if(r.w==null) { e=r.ye; ce=r.ce; ne=0; }
						 else {
						 	e=0;
						 	if(nt==0) { ne=r.w.w.yd; nce=r.w.w.cd; }
						 	else { ne=r.w.yd; nce=r.w.cd; }
						 }
						 r=r.n;
						 break;
				case S : if(r.s==null) { nt=r.yd; nct=r.cd; }
						 else nt=0;
						 if(r.e==null) { e=r.yd; ce=r.cd; ne=0; }
						 else {
						 	e=0;
						 	if(nt==0) { ne=r.e.e.yd; nce=r.e.e.cd; }
						 	else { ne=r.e.yd; nce=r.e.cd; }
						 }
						 if(r.w==null) { d=r.ye; cd=r.ce; nd=0; }
						 else {
						 	d=0;
						 	if(nt==0) { nd=r.w.w.yd; ncd=r.w.w.cd; }
						 	else { nd=r.w.yd; ncd=r.w.cd; }
						 }
						 r=r.s;
						 break;
				case E : if(r.e==null) { nt=r.yd; nct=r.cd; }
						 else nt=0;
						 if(r.s==null) { d=r.yd; cd=r.cd; nd=0; }
						 else {
						 	d=0;
						 	if(nt==0) { nd=r.s.s.yd; ncd=r.s.s.cd; }
						 	else { nd=r.s.yd; ncd=r.s.cd; }
						 }
						 if(r.n==null) { e=r.ye; ce=r.ce; ne=0; }
						 else {
						 	e=0;
						 	if(nt==0) { ne=r.n.n.yd; nce=r.n.n.cd; }
						 	else { ne=r.n.yd; nce=r.n.cd; }
						 }
						 r=r.e;
						 break;
				case W : if(r.w==null) { nt=r.ye; nct=r.ce; }
						 else nt=0;
						 if(r.s==null) { e=r.yd; ce=r.cd; ne=0; }
						 else {
						 	e=0;
						 	if(nt==0) { ne=r.s.s.ye; nce=r.s.s.ce; }
						 	else { ne=r.s.ye; nce=r.s.ce; }
						 }
						 if(r.n==null) { d=r.ye; cd=r.ce; nd=0; }
						 else {
						 	d=0;
						 	if(nt==0) { nd=r.n.n.ye; ncd=r.n.n.ce; }
						 	else { nd=r.n.ye; ncd=r.n.ce; }
						}
						 r=r.w;
						 break;
			}
			listOfroom.addElement( new roomvar( n, (c[0].cnt-1)*Pasz, e, d, le,
									ld, ne, nd, nt, ce, cd, nce, ncd, nct) );
			if(n==0 && c[0].cnt==Dim && d>0) {
				remCt=Dim;
				remY=d;
				remC=cd;
			}
		}
	}
	while(listOfroom.size()>0) {
		roomvar tv=(roomvar)(listOfroom.elementAt(listOfroom.size()-1));
		room(tv.base, tv.offst, tv.e, tv.d, tv.le, tv.ld, tv.ne, tv.nd, tv.nt,
				tv.ce, tv.cd, tv.nce, tv.ncd, tv.nct);
		listOfroom.removeElementAt(listOfroom.size()-1);
	}
	if(remCt>0 && remCt<Dim) {
		polyX[0]=conv3d2d(Lx_4,1)+width2;
		polyX[1]=conv3d2d(Lx_4,(remCt+1)*Pasz)+width2;
		polyX[2]=polyX[1];
		polyX[3]=polyX[0];
		polyY[0]=conv3d2d(Ly,1)+height2;
		polyY[1]=conv3d2d(Ly,(remCt+1)*Pasz)+height2;
		polyY[2]=conv3d2d(-remY,(remCt+1)*Pasz)+height2;
		polyY[3]=conv3d2d(-remY,1)+height2;
		backg.setColor(remC);
		backg.fillPolygon(polyX,polyY,4);
	}
	if(remCt>0)	remCt--;
	c[0].clone(lc);
	}
	public void room(int base, int offst, int e, int d, int le, int ld, int ne,
			int nd, int nt, Color ce, Color cd, Color nce, Color ncd, Color nct) {
	int t=Lenz+base*Lenz-offst, temp;
	pix3d aux=new pix3d();
	pix3d ale=new pix3d();
	pix3d ald=new pix3d();
	pix3d aue=new pix3d();
	pix3d aud=new pix3d();
	pix3d nle=new pix3d();
	pix3d nld=new pix3d();
	pix3d nue=new pix3d();
	pix3d nud=new pix3d();
	int[] polyX=new int[4];
	int[] polyY=new int[4];
	ale.x=-Lx_24;
	ale.y=Ly;
	ale.z=ald.z=aue.z=t;
	ald.x=Lx_4;
	ald.y=Ly;
	aue.x=-Lx_24;
	aue.y=-e;
	aud.x=Lx_4;
	aud.y=-d;
	aud.z=t;
	ale.clone(nle);
	ald.clone(nld);
	aue.clone(nue);
	aud.clone(nud);
	nle.z=nld.z=nue.z=nud.z=t+Lenz;
	if(e!=0) {
		if(e>le) {
			ale.clone(aux);
			if(le==0) aux.y=Ly;
			else aux.y=aue.y+e-le;
			polyX[0]=conv3d2d(aue.x,aue.z)+width2;
			polyY[0]=conv3d2d(aue.y,aue.z)+height2;
			polyX[1]=polyX[0];
			polyY[1]=conv3d2d(aux.y,aux.z)+height2;
			aue.clone(aux);
			aux.x-=200;
			polyX[2]=conv3d2d(aux.x,aux.z)+width2;
			polyY[2]=polyY[1];
			polyX[3]=polyX[2];
			polyY[3]=polyY[0];
			backg.setColor(ce);
			backg.fillPolygon(polyX,polyY,4);
		}
		polyX[0]=conv3d2d(ale.x,ale.z)+width2;
		polyX[1]=conv3d2d(nle.x,nle.z)+width2;
		polyX[2]=conv3d2d(nue.x,nue.z)+width2;
		polyX[3]=conv3d2d(aue.x,aue.z)+width2;
		polyY[0]=conv3d2d(ale.y,ale.z)+height2;
		polyY[1]=conv3d2d(nle.y,nle.z)+height2;
		polyY[2]=conv3d2d(nue.y,nue.z)+height2;
		polyY[3]=conv3d2d(aue.y,aue.z)+height2;
		backg.setColor(ce);
		backg.fillPolygon(polyX,polyY,4);
	}
	else if(nt!=0 || (nt==0 && base<5)) {
		temp=(nt==0)? 200 : 0;
		polyX[0]=conv3d2d(nue.x-temp,nue.z)+width2;
		polyX[1]=polyX[0];
		polyX[2]=conv3d2d(nue.x-300-temp,nue.z)+width2;
		polyX[3]=polyX[2];
		polyY[0]=conv3d2d(-ne,nue.z)+height2;
		polyY[1]=conv3d2d(nle.y,nle.z)+height2;
		polyY[2]=polyY[1];
		polyY[3]=polyY[0];
		backg.setColor(nce);
		backg.fillPolygon(polyX,polyY,4);
	}
	if(d!=0) {
		if(d>ld) {
			ald.clone(aux);
			if(ld==0) aux.y=Ly;
			else aux.y=aud.y+d-ld;
			polyX[0]=conv3d2d(aud.x,aud.z)+width2;
			polyY[0]=conv3d2d(aud.y,aud.z)+height2;
			polyX[1]=polyX[0];
			polyY[1]=conv3d2d(aux.y,aux.z)+height2;
			aud.clone(aux);
			aux.x+=200;
			polyX[2]=conv3d2d(aux.x,aux.z)+width2;
			polyY[2]=polyY[1];
			polyX[3]=polyX[2];
			polyY[3]=polyY[0];
			backg.setColor(cd);
			backg.fillPolygon(polyX,polyY,4);
		}
		polyX[0]=conv3d2d(ald.x,ald.z)+width2;
		polyX[1]=conv3d2d(nld.x,nld.z)+width2;
		polyX[2]=conv3d2d(nud.x,nud.z)+width2;
		polyX[3]=conv3d2d(aud.x,aud.z)+width2;
		polyY[0]=conv3d2d(ald.y,ald.z)+height2;
		polyY[1]=conv3d2d(nld.y,nld.z)+height2;
		polyY[2]=conv3d2d(nud.y,nud.z)+height2;
		polyY[3]=conv3d2d(aud.y,aud.z)+height2;
		backg.setColor(cd);
		backg.fillPolygon(polyX,polyY,4);
	}
	else if(nt!=0 || (nt==0 && base<3)) {
		temp=(nt==0)? 200 : 0;
		polyX[0]=conv3d2d(nud.x+temp,nud.z)+width2;
		polyY[0]=conv3d2d(-nd,nud.z)+height2;
		polyX[1]=polyX[0];
		polyY[1]=conv3d2d(nld.y,nld.z)+height2;
		polyX[2]=conv3d2d(nud.x+300+temp,nud.z)+width2;
		polyY[2]=polyY[1];
		polyX[3]=polyX[2];
		polyY[3]=polyY[0];
		backg.setColor(ncd);
		backg.fillPolygon(polyX,polyY,4);
	}
	if(nt!=0) {
		polyX[0]=conv3d2d(nle.x,nle.z)+width2;
		polyX[1]=conv3d2d(nue.x,nue.z)+width2;
		polyX[2]=conv3d2d(nud.x,nud.z)+width2;
		polyX[3]=conv3d2d(nld.x,nld.z)+width2;
		polyY[0]=conv3d2d(Ly,nle.z)+height2;
		polyY[1]=conv3d2d(-nt,nue.z)+height2;
		polyY[2]=conv3d2d(-nt,nud.z)+height2;
		polyY[3]=conv3d2d(Ly,nld.z)+height2;
		backg.setColor(nct);
		backg.fillPolygon(polyX,polyY,4);
	}
	}
	public int conv3d2d(int v, int z) {
		double m=(double)v/(double)z;
		return (int)(m*PRF);
	}
}
```

