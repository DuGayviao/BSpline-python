#                           Universidade Tecnológica Federal do Paraná                              # 
#                                        Computação Gráfica                                         #
#                                      Eduarda Simonis Gavião                                       #
#                                          Fernanda Azevedo                                         #
#________________________________________________###________________________________________________#
# |Para iniciar a coleta de coordenadas abra o menu, clicando com o scroll do mouse                 |
# |Utilizando o mouse:                                                                              |
# |                  Para setar os pontos basta clicar na área dimensionada                         |
# |                  Para criar uma nova curva clique com o botão direito                           |
# |                  Curvas BSpline trabalham de 4 á 9 pontos                                       |
# |                  Após 9 pontos uma nova curva é criada                                          | 
# |                 Com menos de 4 pontos não é possivel criar uma nova curva                       |
# |Utilizando o teclado:                                                                            | 
# |                  Primeiramente insira o número de pontos que sua curva terá                     |  
# |                  Para setar os pontos, digite as coordenadas de x e y                           | 
# |                  Para criar uma nova curva abra o menu novamente e selecione a opção de entrada |
# |Teclas especiais:                                                                                |
# |                   C ou c->a tela é limpa                                                        |
# |                   T ou t -> abre a entrada pelo teclado                                         |
# |                   M ou m -> abre a entrada pelo mouse                                           |
# |                   Esc -> A tela é deletada e o sistema fechado                                  |
#____________________________________________________________________________________________________

#Chamada das bibliotecas
from itertools import count
from OpenGL.GL import *
from OpenGL.GLU import *
from OpenGL.GLUT import *
from scipy import interpolate
import matplotlib.pyplot as plt
import numpy as np 

w, h =500,500 #definindo altura e largura (universo) tamanho wide screen
#definindo os vetores e váriaveis globais
win=1
vetX=[]
vetY=[]
curves=0
matrixFlag = np.empty((500,500))
print(curves)

#cria e plota a curva bspline
def Bspline(vetX,vetY):
    global curves
    l=len(vetX)  
    curves+=1
    
    if curves <= 5:   #enquanto o número de curvas for menor ou igual a 5 novas curvas serão criadas   
     
     t=np.linspace(0,1,l-2,endpoint=True)
     t=np.append([0,0,0],t)
     t=np.append(t,[1,1,1])

     tck=[t,[vetX,vetY],3]
     u3=np.linspace(0,1,(max(l*2,70)),endpoint=True)
     out = interpolate.splev(u3,tck)

     #pontos retornados da função
     x=out[0] #coordenadas de x
     y=out[1] #coordenadas de y
     
     cores(curves) #chama a função de cores que seta uma cor determinada de acordo com o número da curva
     glBegin(GL_LINES) #inicia as retas
     for i in range(0, len(y)-1,1): #for de 0 até o fim do vetor
         j=i+1 #j+1 para o próximo ponto (variável auxiliar)
         #declaração dos pontos para traçado da reta 
         #2 pontos para a definição de cada reta
         glVertex2f(x[i],y[i]) #passa a coordenada do ponto atual
         glVertex2f(x[j],y[j]) #passa a coordenada do ponto posterior    
           
     glEnd()      
     
    else: #caso contrário não serão mais plotadas curvas
        print('Quantidade máxima de curvas antigida')

#definindo as cores das curvas  
def cores(i):
    if (i==1): #curva 1
        cor=glColor3f(0, 1, 0) #define a cor dos pontos
    if(i==2): #curva 2
        cor=glColor3f(0, 0, 1) #define a cor dos pontos
    if(i==3): #curva 3
        cor=glColor3f(1, 0, 0) #define a cor dos pontos
    if(i==4): #curva 4
        cor=glColor3f(1, 0, 1) #define a cor dos pontos
    if(i==5): #curva 5
        cor=glColor3f(1, 1, 0) #define a cor dos pontos

def MenuTeclado(options,x,y):
    global curves
    global win
    if options == b't':
        teclado()   
    if options == b'T':
        teclado()      
    if options == b'm':
        glutMouseFunc(mouse)   
    if options == b'M':
        glutMouseFunc(mouse)  
    if options == b'C':
        #Limpa a Tela e habilita os valores glClearColor
        curves=0
        glutDestroyWindow(win)
        main()
    if options == b'c':
        #Limpa a Tela e habilita os valores glClearColor
        curves=0
        glutDestroyWindow(win)
        main()
       
    if options == b'\x1b':
        glutDestroyWindow(win)
        sys.exit()
    glutPostRedisplay()
        
#definindo entradas com os valores obtidos pelo teclado
def teclado():
    count=1
    n=int(input("Digite a quantidade de pontos que deseja ")) #limita a quantidade de pontos
    if n<=3 or n>9: #se for maior ou menor que os limites deve ser digitado novamente
        print("Digite uma quantidade de pontos válida [4-9] ")
        teclado()
    while count<=n:         #equanto o contador for menor q o número de pontos, se insere os pontos
      x= float(input('Digite a coordenada de X: '))      
      y= float(input('Digite a coordenada de Y: '))
      tec(x,y)#passa os pontos para a plotagem no plano
      if matrixFlag[int(x)][int(y)]==0:            #verifica a existencia de pontos iguais
            vetX.append(x) #armazena os pontos nos vetores
            vetY.append(y)   
            matrixFlag[int(x)][int(y)]=1    
      else:
         print('Coordenada já utilizada utilize outro valor')         
      count+=1            
    Bspline(vetX,vetY)   #chamada da função da spline
    
    if count>n: # se maior que o número de pontos setados limpa os vetores
        vetX.clear()
        vetY.clear()
        
    glFlush()
    glutPostRedisplay()

#função para a plotagem dos pontos     
def tec(x,y):
    glBegin(GL_POINTS) #registra os pontos
    glVertex2f(x,y) #coordenadas x e y         
    glEnd()
    glFlush()
    glutPostRedisplay()

#definindo função para utilização do mouse 
def mouse ( button,st,mouseX,mouseY):
    
    x = mouseX #define x como sendo as coordenadas de click x do mouse        
    y = h-mouseY #define y como sendo as coordenadas de click y do mouse (-720) pois considera o ponto de origem o canto inferior esquerdo
          
    if button==GLUT_LEFT_BUTTON and st==GLUT_DOWN : #verifica se o botão clicado é o ESQUERDO , St é o estado se foi pressionado ou liberado
        glBegin(GL_POINTS) #registra os pontos
        
        if matrixFlag[int(x)][int(y)]==0:
             vetX.append(x)
             vetY.append(y)   
             matrixFlag[x][y]=1    
        else:
         print('Coordenada já utilizada utilize outro valor')               
        glVertex2f(x,y) #coordenadas x e y         
        glEnd() 
        
         
        if len(vetY)%9 ==0: #VARIÁVEL DE CONTROLE 
            Bspline(vetX,vetY)
        if len(vetY) >9:    
            vetX.clear()
            vetY.clear()
    if button==GLUT_RIGHT_BUTTON and st==GLUT_DOWN:  
        if len(vetY)<=3:
          print('Quantidade mínima de pontos para B-Spline não atingida insira mais pontos')
        else:
          Bspline(vetX,vetY)
          vetX.clear()
          vetY.clear()
    glFlush()
    glutPostRedisplay()

#função para o traçado e demarcação dos eixos   
def desenhaEixos():
    
    glColor3f(0.3,0.3,0.3)#define a cor da linha
    glLineWidth(0.5) #define a expessura da linha 
    glBegin(GL_LINES)
    for i in range(0,w,25):
        glVertex2i(i,-500);  glVertex2i(i,500); #define as delimitações do x
    for j in range(0,h,25):
        glVertex2i(-500,j);  glVertex2i(500,j); #define as delimitações do y
    glEnd()

    glColor3f(1,1,1) #define a cor da linha
    glLineWidth(3.0) #define a expessura da linha 
    glBegin(GL_LINES) #plota as linhas após 2 glVertex
    glVertex2i(0,0);  glVertex2i(0,500); #eixo y positivo
    glVertex2i(0,0);  glVertex2i(0,-500); #eixo y negativo
    glVertex2i(0,0);  glVertex2i(500,0); #eixo x positivo
    glVertex2i(0,0);  glVertex2i(-500,0); #eixo x negativo
    glEnd();
    glutPostRedisplay()

#função para fechar
def exit():
    print('Você fechou o menu')
    sys.exit()

#casos do menu
def entrada(case):
    if case==1:
        glutMouseFunc(mouse)
    if case==2:
        return teclado()
    if case==0:
        return exit()
    glutPostRedisplay()

#criação das opções do menu 
def Menu():
    glutCreateMenu(entrada) #inicializando o menu e passado o mapeamento das opções
    glutAddMenuEntry("Utilizar Mouse",1) #primeira opção coordenadas do mouse
    glutAddMenuEntry("Utilizar Teclado",2) #segunda opção coordenadas do teclado
    glutAddMenuEntry("Fechar",0)  #terceira opção fechar
    glutAttachMenu(GLUT_MIDDLE_BUTTON) #seta para o scroll do mouse

#faz o gerenciamento do menu na tela 
def GerenciaMouse(button, st,x,y):
    if button==GLUT_MIDDLE_BUTTON and st==GLUT_DOWN: #seta para o scroll do mouse pressionado 
        Menu()
    glutPostRedisplay()

#definições de parametros de cor, tamanho da janela, dos pontos, e das curvas
def iterate():
   glViewport(0, 0, w,h)
   glMatrixMode(GL_PROJECTION)
   glLoadIdentity()
   glOrtho(0.0, w, 0.0, h, 0.0, 1.0)
   glMatrixMode (GL_MODELVIEW)
   glLoadIdentity()
   
def Janela():
   glPointSize(6)#define o tamanho dos pontos
   glLoadIdentity() #Redefine as posições
   desenhaEixos()
   iterate()# chama a função de iteração
   glutSwapBuffers()

#função main para a chamada das funções necessárias 
def main():   
    global curves 
    global win

    glutInit()
    glutInitDisplayMode(GLUT_RGBA)  #define modo de exibição colorido, vermelho, verde, azul e alpha
    glutInitWindowSize(w,h) #seta as dimensões da janela
    glutInitWindowPosition(0,0) #seta a posição inicial da janela
    win= glutCreateWindow("PyOpenGL Curvas") # seta o titulo da imagem
    int(win)
    glutDisplayFunc(Janela)
    glutIdleFunc(Janela) #mantem a janela aberta
    glutMouseFunc(GerenciaMouse)
    glutKeyboardFunc(MenuTeclado)
    glutMainLoop() #mantém a janela setada em loop 
    

if __name__ == "__main__":
    main()
