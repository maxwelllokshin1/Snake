# Snake
Simple snake game developed within one week, Game was created through Java using Eclipse, Got a better understanding of Java's GUI

### Link to download files, click on GIF

[![snakeGIF](https://github.com/user-attachments/assets/9693614a-92c6-4f93-85d5-d62e644429f7)](https://github.com/maxwelllokshin1/javaProjects/blob/main/Jar%20File%20games/snek.zip)



**JAVA**

## Code files written here:

### Class window
```sh
package mainWindow;

import java.awt.Dimension;

import javax.swing.JFrame;

public class window {
	public static JFrame window;
	public static int scale = 25;
	public static int boardWidth = scale * 25;
	public static int boardHeight = scale * 25;
	public static void main(String[] args)
	{
		window = new JFrame("Snake");
		
		window.setMinimumSize(new Dimension(boardWidth, boardHeight));
		window.setResizable(false);
		window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		
		mechanics m = new mechanics();
		window.add(m);
		window.pack();
		
		window.setLayout(null);
		window.setLocationRelativeTo(null);
		window.setVisible(true);
	}
}

```


### Class keyH
```sh
package mainWindow;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

import mainWindow.mechanics.Player;

public class keyH implements KeyListener{

	mechanics m;
	
	public keyH(mechanics m)
	{
		this.m = m;
	}
	
	
	@Override
	public void keyTyped(KeyEvent e) { }

	@Override
	public void keyPressed(KeyEvent e) {
		
		int keyCode = e.getKeyCode();
		if(!m.gameOver)
		{
			if(keyCode == KeyEvent.VK_UP){ m.upPressed = true; m.downPressed = false; m.rightPressed = false; m.leftPressed = false;}
			else if(keyCode == KeyEvent.VK_LEFT){ m.upPressed = false; m.downPressed = false; m.rightPressed = false; m.leftPressed = true;}
			else if(keyCode == KeyEvent.VK_RIGHT){ m.upPressed = false; m.downPressed = false; m.rightPressed = true; m.leftPressed = false;}
			else if(keyCode == KeyEvent.VK_DOWN){ m.upPressed = false; m.downPressed = true; m.rightPressed = false; m.leftPressed = false;}
		}else
		{
			if(keyCode == KeyEvent.VK_ENTER){ m.resetGame();}
		}
	}

	@Override
	public void keyReleased(KeyEvent e) { }

}

```

### Class mechanics
```sh
package mainWindow;

import java.awt.Color;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.Graphics;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Iterator;
import java.util.LinkedList;

import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.Timer;

import mainWindow.mechanics.Player;


public class mechanics extends JPanel implements ActionListener{
	public int screenWidth = window.boardWidth;
	public int screenHeight = window.boardHeight;
	public boolean upPressed, leftPressed, downPressed, rightPressed;
	public Timer gameLoop;

    public int FPS = 120;
    
    keyH keyHandler;
    int pWidth = window.scale, pHeight = window.scale;
    int px = window.scale*5, py = window.scale*6;
    
    final int rWidth = window.scale, rHeight = window.scale;
    
    boolean captured = true;
    LinkedList<Player> snake;
        
    double velocity = 25.0;
    double dx = 0, dy= 0;
    
    int score = 0;
	
    boolean gameOver = false;
    
    int movingCounter = 0;
    int movingSpeed = 10;
    
	public mechanics()
	{
        setPreferredSize(new Dimension(screenWidth, screenHeight));
        setBackground(Color.BLACK);
        setFocusable(true);
        keyHandler = new keyH(this);
        addKeyListener(keyHandler);
        
        snake = new LinkedList<>();
        snake.add(new Player(px, py));
        
        gameLoop = new Timer(1000/FPS, this);
        gameLoop.start();
	}
	
	@Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        drawGameScreen(g);
    }
	
	public void drawGameScreen(Graphics g)
	{
		if(gameOver)
		{
			drawGameOverScreen(g);
		}
		g.setColor(new Color(255,255,255,255));
		g.setFont(getFont().deriveFont(30F));
		g.drawString("SCORE: " + score, window.scale, window.scale*2);
		
		if(r != null)
		{	
			g.setColor(new Color(255,0,0,255));
			g.fillRect(r.x, r.y, r.width, r.height);
		}
		
		for(Player segment : snake)
		{
			g.setColor(new Color(0,255,0,255));
			g.fillRect(segment.x, segment.y, segment.width, segment.height);
		}
	}
	
	public void drawGameOverScreen(Graphics g)
	{
		g.setColor(new Color(255,255,255,255));
		g.setFont(getFont().deriveFont(50F));
		g.drawString("[GAME OVER]", getXforCenteredText("[GAME OVER]", g), screenHeight/2);
		
		gameLoop.stop();
	}
	
	public void move()
	{
		movingCounter++;
		placeRed();
		if(upPressed) {dy = -velocity; dx = 0;}
		else if(downPressed) {dy = velocity; dx = 0; }
		else if(leftPressed) {dx = -velocity; dy = 0; }
		else if(rightPressed) {dx = velocity; dy = 0; }
		
		Player head = snake.getFirst();
		if(upPressed || downPressed) {head.x = closestCoord(head.x);}
		else if(leftPressed || rightPressed) {head.y = closestCoord(head.y);}

		if(movingCounter >= movingSpeed)
		{
			movingCounter = 0;
			moveSnake();
		}
		
		if(collision(snake.getFirst(),r))
		{
			r = null;
			addSegments(4);
			score += 1;
		}
	}
	
	public void moveSnake()
	{
		LinkedList<Player> newSnake = new LinkedList<>();
		
		Player head = snake.getFirst();
		int headX = head.x + (int) dx;
		int headY = head.y + (int) dy;
		
		Player newHead = new Player(headX, headY);
		
		newSnake.add(newHead);
		
		for(int i = 0; i<snake.size()-1; i++)
		{
			Player currentSegment = snake.get(i);

	        // Add the updated segment to the new snake
	        newSnake.add(currentSegment);
		}
		
		snake = newSnake;
		
		for(Player segment : snake)
		{
			if(upPressed || downPressed) {segment.x = closestCoord(segment.x);}
			else if(leftPressed || rightPressed) {segment.y = closestCoord(segment.y);}
		}
		
		checkSnakeCollision();
		
		if(gameOver)
		{
			
		}
	}
	
	public void addSegments(int count)
	{
		Player lastSegment = snake.getLast();
		int lastX = lastSegment.x;
		int lastY = lastSegment.y;
		for(int i = 0; i<count; i++)
		{
			snake.add(new Player(lastX, lastY));
		}
	}
	
	public int closestCoord(int coord)
	{
		return Math.round(coord / 25.0f) * window.scale;
	}
	
	Player r;
	
	public void placeRed()
	{
		if(r == null)
		{
			int randX = (int) (Math.random() * (screenWidth/window.scale)) * window.scale;
			int randY = (int) (Math.random() * (screenHeight/window.scale)) * window.scale;
			r = new Player(randX, randY);
		}
	}
	
	public void checkSnakeCollision()
	{
		Player head = snake.getFirst();
		if(head.x > screenWidth-(head.width) || 
		   head.y > screenHeight-(head.height) ||
		   head.x < 0 || 
		   head.y < 0){
			gameOver = true;
		}
		
		for(Player segment : snake)
		{
			if(segment != head)
			{
				if(collision(head, segment))
				{
					gameOver = true;
				}	
			}
		}
	}
	
	public boolean collision(Player p, Player r)
	{
		return p.x < r.x + r.width &&
			   p.x + p.width > r.x &&
			   p.y < r.y + r.height &&
			   p.y + p.height > r.y;
	}
	
	public int getXforCenteredText(String text, Graphics g) {
        int length = (int) g.getFontMetrics().getStringBounds(text, g).getWidth();
        return screenWidth / 2 - length / 2; // Center the text horizontally
    }
	
	 // Reset the game
    public void resetGame() {
        gameLoop.stop();

        // Clear all rocks and reset score, player position, etc.
        snake.clear();
        snake.add(new Player(px, py));
        score = 0;
        gameOver = false;
        
        upPressed = false;
        downPressed = false;
        leftPressed = false;
        rightPressed = false;
        dy = 0;
        dx = 0;

        r = null;
        // Restart timers
        gameLoop.start();
    }
	
	public class Player{
		int x,y;
		int width = pWidth,height = pHeight;
		public Player(int x, int y)
		{
			this.x = x;
			this.y = y;
		}
	}
	
	@Override
	public void actionPerformed(ActionEvent e) {
		move();
		repaint();
	}
}

```

