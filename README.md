# tank

#导入pygame模块
import pygame, time, random
from pygame.sprite import Sprite



SCREEN_WIDTH = 700
SCREEN_HEIGHT = 500
BG_COLOR = pygame.Color(0, 0, 0)
COLOR_RED = pygame.Color(255, 255, 255)
TEXT_COLOR = pygame.Color(255, 0, 0)


class BaseItem(Sprite):                         #定义一个基类
    def __init__(self, color, width, height):
        pygame.sprite.Sprite.__init__(self)


class MainGame():
    window = None
    my_tank = None
    enemyTankList = []       #存储敌方坦克的列表
    enemyTankCount = 7        #敌方坦克的数量
    myBulletList = []         #存储我方子弹列表
    enemyBulletList = []      #存储敌方子弹的列表
    explodeList = []          #存储爆炸效果的列表
    wallList = []             #存储墙壁的列表


    def __init__(self):
        pass

    def startGame(self):
        pygame.display.init()                        #加载主窗口，初始化窗口
        MainGame.window = pygame.display.set_mode([SCREEN_WIDTH, SCREEN_HEIGHT]) #设置窗口的大小以及显示
        self.createMyTank()                          #初始化我方坦克
        self.createEnemyTank()                       #初始化敌方坦克，并将敌方坦克加入到列表中
        self.createWall()                            #初始化墙壁
        pygame.display.set_caption('坦克大战')        #设置窗口标题
        while True:
            time.sleep(0.04)
            MainGame.window.fill(BG_COLOR)           #给窗口设置填充色
            self.getEvent()                          #获取事件
            MainGame.window.blit(self.getTextSurface('敌方坦克剩余数量%d'%len(MainGame.enemyTankList)), (10, 10))

            if MainGame.my_tank and MainGame.my_tank.live:
                MainGame.my_tank.displayTank()        #判断我方坦克是否活着，并调用坦克显示方法
            else:                                     #否则删除我方坦克
                del MainGame.my_tank
                MainGame.my_tank = None


            self.blitEnemyTank()                     #循环遍历敌方坦克，展示敌方坦克
            self.blitMyBullet()                      #循环遍历显示我方坦克的子弹
            self.blitEnemyBullet()                   #循环遍历敌方子弹列表，展示敌方子弹
            self.blitExplode()                       #循环遍历爆炸列表，展示爆炸效果
            self.blitWall()                          #循环遍历，展示墙壁
            if MainGame.my_tank and MainGame.my_tank.live:
                if not MainGame.my_tank.stop:             #调用移动方法，坦克开关开启才可以移动
                    MainGame.my_tank.move()
                    MainGame.my_tank.hitWall()            #检测我方坦克是否与墙壁发生碰撞
                    MainGame.my_tank.myTank_hit_enemyTank()  #检测我方坦克是否与敌方坦克发生碰撞


            pygame.display.update()


    def blitWall(self):                              #循环遍历墙壁，展示墙壁
        for wall in MainGame.wallList:
            if wall.live:
                wall.displayWall()
            else:
                MainGame.wallList.remove(wall)

    def createWall(self):                            #初始化墙壁
        for i in range(6):
            wall = Wall(i*130, 220)
            MainGame.wallList.append(wall)



    def createMyTank(self):                           #创建我方坦克的方法
        MainGame.my_tank = MyTank(350, 350)
        music = Music('img/start.wav')                #创建音乐对象
        music.play()

    def createEnemyTank(self):                         #初始化敌方坦克，并加入敌方坦克列表中
        top = 100
        for i in range(MainGame.enemyTankCount):
            left = random.randint(0, 600)
            speed = random.randint(1, 4)
            enemy = EnemyTank(left, top, speed)
            MainGame.enemyTankList.append(enemy)

    def blitExplode(self):                            #循环展示爆炸效果
        for explode in MainGame.explodeList:
            if explode.live:                          #判断是否活着
                explode.displayExplode()              #展示
            else:                                     #否则从列表里移除
                MainGame.explodeList.remove(explode)

    def blitEnemyTank(self):                          #循环遍历敌方坦克列表，展示敌方坦克
        for enemyTank in MainGame.enemyTankList:
            if enemyTank.live:                        #判断敌方坦克是否活着
                enemyTank.displayTank()
                enemyTank.randMove()
                enemyTank.hitWall()                   #检测是否与墙壁发生碰撞
                if MainGame.my_tank and MainGame.my_tank.live:
                    enemyTank.enemyTank_hit_myTank()  #检测敌方坦克是否与我方坦克发生碰撞

                enemyBullet = enemyTank.shot()            #发射子弹
                if enemyBullet:                           #敌方子弹是否为None
                    MainGame.enemyBulletList.append(enemyBullet) #将敌方子弹存储到敌方子弹列表中
            else:
                MainGame.enemyTankList.remove(enemyTank)   #若是不活着就从列表中移除

    def blitMyBullet(self):                            #循环遍历我方子弹存储列表
        for myBullet in MainGame.myBulletList:
            if myBullet.live:                          #判断当前子弹是否是活着状态，如果是则进行显示以及移动
                myBullet.displayBullet()
                myBullet.move()
                myBullet.myBullet_hit_enemyTank()      #调用检测我方子弹是否与敌方坦克发生碰撞
                myBullet.hitWall()                     #检测我方子弹是否与墙壁碰撞
            else:                                      #否则从列表中删除
                MainGame.myBulletList.remove(myBullet)

    def blitEnemyBullet(self):                          #循环遍历敌方子弹列表，展示敌方子弹
        for enemyBullet in MainGame.enemyBulletList:
            if enemyBullet.live:                         #判断敌方子弹是否活着
                enemyBullet.displayBullet()
                enemyBullet.move()
                enemyBullet.enemyBullet_hit_myTank()     #调用敌方子弹与我方坦克碰撞的方法
                enemyBullet.hitWall()                    #检测敌方子弹是否与墙壁碰撞

            else:
                MainGame.enemyBulletList.remove(enemyBullet)


    def getTextSurface(self, text):                         #左上角的文字绘制
        pygame.font.init()                                  #初始化字体模块
        font = pygame.font.SysFont('kaiti', 10)             #获取字体font对象
        textSurface = font.render(text, True, TEXT_COLOR)    #绘制文字信息
        return textSurface

    def endGame(self):
        print('谢谢使用！')
        exit()

    def getEvent(self):
        eventList = pygame.event.get()          #获取素所有事件
        for event in eventList:                 #遍历事件，并判断是点击关闭按钮还是按键盘
            if event.type == pygame.QUIT:       #如果点击关闭按钮，就关闭窗口
                self.endGame()
            if event.type == pygame.KEYDOWN:    #如果按下键盘上的键，就进行如下判断

                if not MainGame.my_tank:        #判断是否按下ESC键，若是则重生
                    if event.key == pygame.K_ESCAPE:
                        self.createMyTank()

                if MainGame.my_tank and MainGame.my_tank.live:
                    #进行上下左右的判断
                    if event.key == pygame.K_LEFT:
                        MainGame.my_tank.direction = 'L'
                        MainGame.my_tank.stop = False    #修改坦克状态开关
                        print('按下左键，坦克左移')
                    elif event.key == pygame.K_RIGHT:
                        MainGame.my_tank.direction = 'R'
                        MainGame.my_tank.stop = False    #修改坦克状态开关
                        print('按下左键，坦克右移')
                    elif event.key == pygame.K_UP:
                        MainGame.my_tank.direction = 'U'
                        MainGame.my_tank.stop = False    #修改坦克状态开关
                        print('按下左键，坦克上移')
                    elif event.key == pygame.K_DOWN:
                        MainGame.my_tank.direction = 'D'
                        MainGame.my_tank.stop = False    #修改坦克状态开关
                        print('按下左键，坦克下移')
                    elif event.key == pygame.K_SPACE:
                        print('发射子弹')
                        if len(MainGame.myBulletList) < 3:   #我方子弹列表小于3的时候才可以创建
                            myBullet = Bullet(MainGame.my_tank)    #创建我方坦克发射的子弹
                            MainGame.myBulletList.append(myBullet)
                            music = Music('img/hit.wav')           #加入发射子弹音效
                            music.play()

            if event.type == pygame.KEYUP:           #松开方向键，坦克停止移动，修改坦克开关
                if (event.key == pygame.K_UP
                        or event.key == pygame.K_DOWN
                        or event.key == pygame.K_LEFT
                        or event.key == pygame.K_RIGHT):
                    if MainGame.my_tank and MainGame.my_tank.live:
                        MainGame.my_tank.stop = True



class Tank(BaseItem):
    def __init__(self, left, top):                #距离左边left，距离上边top
        self.images = {
            'U': pygame.image.load('img/p1tankU.gif'),
            'D': pygame.image.load('img/p1tankD.gif'),
            'L': pygame.image.load('img/p1tankL.gif'),
            'R': pygame.image.load('img/p1tankR.gif'),
        }
        self.direction = 'L'                       #方向
        self.image = self.images[self.direction]   #根据当前图片方向获取图片surface
        self.rect = self.image.get_rect()          #根据图片获取区域
        self.rect.left = left                      #设置区域的left和top
        self.rect.top = top
        self.speed = 5                            #移动速度
        self.stop = True                          #坦克移动开关
        self.live = True                          #是否活着
        self.oldLeft = self.rect.left             #增加原有坐标
        self.oldTop = self.rect.top

    def move(self):   #移动
        self.oldLeft = self.rect.left  # 移动后记录原始坐标
        self.oldTop = self.rect.top

        if self.direction == 'L':       #判断坦克移动
            if self.rect.left>0:
                self.rect.left -= self.speed
        elif self.direction == 'U':
            if self.rect.top > 0:
                self.rect.top -= self.speed
        elif self.direction == 'D':
            if self.rect.top+self.rect.height<SCREEN_HEIGHT:
                self.rect.top += self.speed
        elif self.direction == 'R':
            if self.rect.left+self.rect.height<SCREEN_WIDTH:
                self.rect.left += self.speed

    def shot(self):    #射击
        return Bullet(self)

    def stay(self):          #坦克停留在原地
        self.rect.left = self.oldLeft
        self.rect.top = self.oldTop

    def hitWall(self):       #检测坦克与墙壁是否发生碰撞，若是，在停在原地
        for wall in MainGame.wallList:
            if pygame.sprite.collide_rect(self, wall):
                self.stay()

    def displayTank(self):   #展示
        self.image = self.images[self.direction]     #获取展示的对象
        MainGame.window.blit(self.image, self.rect)  #调用blit方法展示


#我方坦克
class MyTank(Tank):
    def __init__(self, left, top):
        super(MyTank, self).__init__(left, top)

    def myTank_hit_enemyTank(self):
        for enemyTank in MainGame.enemyTankList:
            if pygame.sprite.collide_rect(self, enemyTank):
                self.stay()


#敌方坦克
class EnemyTank(Tank):
    def __init__(self, left, top, speed):
        super(EnemyTank, self).__init__(left, top)        #调用父类的初始化方法
        self.images = {                                   #加载图片集
            'U': pygame.image.load('img/enemy1U.gif'),
            'D': pygame.image.load('img/enemy1D.gif'),
            'L': pygame.image.load('img/enemy1L.gif'),
            'R': pygame.image.load('img/enemy1R.gif'),
        }
        self.direction = self.randDirection()             #随机生成敌方坦克方向
        self.image = self.images[self.direction]          #获取方向图片
        self.rect = self.image.get_rect()                 #获取区域
        self.rect.left = left                             #对区域位置进行赋值（left）
        self.rect.top = top                               #对区域位置进行赋值（top）
        self.speed = speed                                #速度赋值
        self.flag = True                                  #移动开关
        self.step = 60                                    #增加一个步数变量

    def enemyTank_hit_myTank(self):                       #敌方坦克与我方坦克是否发生碰撞
        if pygame.sprite.collide_rect(self, MainGame.my_tank):
            self.stay()

    def randDirection(self):                              #随机生成敌方坦克方向
        num = random.randint(1, 4)
        if num == 1:
            return 'U'
        if num == 2:
            return 'D'
        if num == 3:
            return 'L'
        if num == 4:
            return 'R'

    def randMove(self):
        if self.step <= 0:
            self.direction = self.randDirection()         #修改方向
            self.step = 60                                #复位步数
        else:
            self.move()
            self.step -= 1                                #让步数递减

    def shot(self):            #重写shot（）
        num = random.randint(1, 100)
        if num < 10:
            return Bullet(self)


#子弹类
class Bullet():
    def __init__(self, tank):
        self.image = pygame.image.load('img/enemymissile.gif')     #加载图片
        self.direction = tank.direction                            #坦克方向决定子弹方向
        self.rect = self.image.get_rect()                            #获取区域
        if self.direction == 'U':
            self.rect.left = tank.rect.left + tank.rect.width / 2 - self.rect.width / 2
            self.rect.top = tank.rect.top - self.rect.height
        elif self.direction == 'D':
            self.rect.left = tank.rect.left + tank.rect.width / 2 - self.rect.width / 2
            self.rect.top = tank.rect.top + tank.rect.height
        elif self.direction == 'L':
            self.rect.left = tank.rect.left - self.rect.width / 2 - self.rect.width / 2
            self.rect.top = tank.rect.top + tank.rect.width/2 - self.rect.width / 2
        elif self.direction == 'R':
            self.rect.left = tank.rect.left + tank.rect.width
            self.rect.top = tank.rect.top + tank.rect.width / 2 - self.rect.width / 2

        self.speed = 6           #子弹的速度
        self.live = True         #子弹的状态，如果碰壁就修改此状态

    def move(self):  # 移动
        if self.direction == 'U':
            if self.rect.top > 0:
                self.rect.top -= self.speed
            else:
                self.live = False          #修改子弹状态
        elif self.direction == 'R':
            if self.rect.left + self.rect.width < SCREEN_WIDTH:
                self.rect.left += self.speed
            else:
                self.live = False          #修改子弹状态
        elif self.direction == 'D':
            if self.rect.top + self.rect.height < SCREEN_HEIGHT:
                self.rect.top += self.speed
            else:
                self.live = False          #修改子弹状态
        elif self.direction == 'L':
            if self.rect.left> 0:
                self.rect.left -= self.speed
            else:
                self.live = False          #修改子弹状态


    def hitWall(self):                     #子弹碰撞墙壁类
        for wall in MainGame.wallList:
            if pygame.sprite.collide_rect(self, wall):
                self.live = False
                wall.hp -= 1
                if wall.hp <=0:
                    wall.live = False

    def displayBullet(self):   #展示
        MainGame.window.blit(self.image, self.rect)   #将图片surface加载到窗口

    def myBullet_hit_enemyTank(self):                 #我方子弹与敌方坦克的碰撞
        for enemyTank in MainGame.enemyTankList:
            if pygame.sprite.collide_rect(enemyTank, self):
                enemyTank.live = False                #修改敌方坦克和我方子弹状态
                self.live = False
                explode = Explode(enemyTank)          #创建爆炸对象
                MainGame.explodeList.append(explode)  #将爆炸对象添加到爆炸列表中

    def enemyBullet_hit_myTank(self):                 #敌方子弹与我方坦克的碰撞
        if MainGame.my_tank and MainGame.my_tank.live:
            if pygame.sprite.collide_rect(MainGame.my_tank, self):
                explode = Explode(MainGame.my_tank)    #创建爆炸对象
                MainGame.explodeList.append(explode)   #将爆炸对象添加到列表里
                self.live = False                      #修改状态
                MainGame.my_tank.live = False



#墙类
class Wall():
    def __init__(self, left, top):
        self.image = pygame.image.load('img/steels.gif')
        self.rect = self.image.get_rect()
        self.rect.left = left
        self.rect.top = top
        self.live = True
        self.hp = 3

    def displayWall(self):   #展示
        MainGame.window.blit(self.image, self.rect)


#爆炸类
class Explode():
    def __init__(self, tank):
        self.rect = tank.rect           #爆炸位置由当前子弹打中位置决定
        self.images = [
            pygame.image.load('img/blast0.gif'),
            pygame.image.load('img/blast1.gif'),
            pygame.image.load('img/blast2.gif'),
            pygame.image.load('img/blast3.gif'),
            pygame.image.load('img/blast4.gif'),
        ]
        self.step = 0
        self.image = self.images[self.step]
        self.live = True

    def displayExplode(self):   #展示爆炸效果
        if self.step < len(self.images):
            self.image = self.images[self.step]         #根据索引获取爆炸对象
            self.step += 1
            MainGame.window.blit(self.image, self.rect) #添加到主窗口
        else:                                           #修改活着的状态
            self.live = False
            self.step = 0


#音乐类
class Music():
    def __init__(self, filename):
        self.filename = filename
        pygame.mixer.init()                                #初始化音乐混合器
        pygame.mixer.music.load(self.filename)             #加载音乐

    def play(self):
        pygame.mixer.music.play()


if __name__ == '__main__':
    MainGame().startGame()
























