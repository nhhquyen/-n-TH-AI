- Link Github
  + https://github.com/nhhquyen/-n-TH-AI.git
  
- Cách quy ước chạy map
  + Vat can : là dòng đọc những tọa độ của vật cản
    • Với số lượng của vật cản (ở dưới chữ Vat can) 
    • Dưới số lượng là những tọa độ của vật cản
  + Cay xang: là dòng đọc những tọa độ cây xăng
    • Với số lượng của cây xăng (ở dưới chữ Cay xang
    • Dưới ố lượng cây xăng là những tọa độ đặt cây xăng
  + Dich den: là dòng đọc tọa độ đích đến
    • Với ở dưới là phần tọa độ của đích đến
  + Nguoi: là vị trí người khi bắt đầu
    • Ở dưới là phần tọa độ của đích
  + Xang: là dòng đọc số lít xăng
    • Phần ở dưới chữ Xang là giá trị lít xăng
    
 - Hướng dẫn cách chạy chương trình
import pygame
import random, time,os
import heapq
import sys
import numpy
from collections import deque

# Khởi tạo pygame()
pygame.init()

# Định nghĩa các màu
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (128, 128, 128)
BLUE = (240,248,255)

#Khởi tạo màn hình width and height, background
size = (500, 500)
screen = pygame.display.set_mode(size)
screen.fill(WHITE)


# Tựa đề 
pygame.display.set_caption("Do an")

#Biến done=True khi nhấn vào dấu "X" tắt màn hình
done = False
People = [] # Tọa độ người
AmountOfImpediment=0 #Biến chứa số lượng vật cản
Impediment=[] #List chứa tọa độ vật cản
AmountOfGasStation=0 #Biến chứa số lượng cây căng
Petrol = 0
GasStation=[] #List chứa tọa độ cây xăng
AmountOfGas=0 #Số lượng lít xăng
Destination=[] # Vị trí đích đến

ImageOfPeople=pygame.transform.scale(pygame.image.load(os.path.join('./DoAnImage','People.png')),(38,38)) # Hình ảnh con người
GasStationImage=pygame.transform.scale(pygame.image.load(os.path.join('./DoAnImage','Gas.PNG')),(38,38))#Hình ảnh cây xăng
Flag=pygame.transform.scale(pygame.image.load(os.path.join('./DoAnImage','flag.PNG')),(38,38)) #Hình ảnh lá cờ đích đến

#Đọc file load lên tất cả lít xăng, địa điểm chứa cây xăng, vật cản.
f=open('./map.txt','r+')
Data=f.readlines()
AmountOfGas=int(Data[0])
for i in range(1,len(Data)):
    if(Data[i].strip().upper()=='VAT CAN'):
        AmountOfImpediment=int(Data[i+1])
        for j in range(i+2,len(Data)):
            if Data[j].upper().strip() !='CAY XANG':
                Impediment.append(Data[j].strip().split())
            else:
                break
    if(Data[i].upper().strip()=='CAY XANG'):
        AmountOfGasStation=int(Data[i+1])
        # Petrol = int(Data[i+2])
        for j in range(i+2,len(Data)):
            if Data[j].upper().strip() !='VAT CAN' and Data[j].upper().strip() !='DICH DEN':
                GasStation.append(Data[j].strip().split())
            else:
                break
    if(Data[i].upper().strip()=='DICH DEN'):
        Destination.append(Data[i+1].strip().split())
    if(Data[i].upper().strip()=='NGUOI'):
        People.append(Data[i+1].strip().split())
    if(Data[i].upper().strip()=='XANG'):
        Petrol = Data[i+1].strip().split()
        break
    
f.close()

class PriorityQueue:
  def __init__(self):
    self.queue = []
  
  def push(self, value, label):
    heapq.heappush(self.queue, (value, label))
  
  def pop(self):
    return heapq.heappop(self.queue)
  
  def is_empty(self):
    # print(self.q)
    return len(self.queue) == 0

class Grid:
  def __init__(self, A, walls):
    self.n = len(A)
    self.m = len(A[0])
    self.A = A
    self.walls = walls

  def in_bounds(self, p):
    x,y  = p
    return x >=0 and y>=0 and x<self.n and y<self.m

  def passable(self, p):
    for wall_pos in self.walls:
      if wall_pos == p:
        return False
    return True
  
  # def petrol(self, g):
  #   for wall_pos in self.gas:
  #     if wall_pos == g:
  #       return False
  #   return True

  def neighbors(self, p):
    x, y = p
    neighbors = [(x+1,y),(x-1,y),(x,y+1),(x,y-1)]
    # print(neighbors)
    valid_neighbors = []
    for pos in neighbors:
      if self.in_bounds(pos) and self.passable(pos):
        valid_neighbors.append(pos)

    # print(valid_neighbors)
    return valid_neighbors

  def draw(self,path=[],done = False):
       while not done:
            #Khi gặp sự kiện nhấn vào dấu "X" sẽ thoát vòng lặp
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    done = True
            #Vẽ bản đồ tại vị trí x=50, y=50
            #Bản đồ với chiều dài là m ô vuông
            for i in range(self.n):
                #Chiều rộng m ô vuông
                for j in range(self.m):
                    # show ra những đường tìm được bởi các thuật toán
                    if (i,j) in path:
                      #Hiển thị cờ đích
                      screen.blit(Flag,(50 + 40 * int(Destination[0][0]), 50 + 40 * int(Destination[0][1])))
                      # Hiển thị người ở hiện tại
                      screen.blit(ImageOfPeople,(50 + 40 * int(People[0][0]), 50 + 40 * int(People[0][1])))#int(People[0][0])
                      # Hiển thị cây xăng
                      screen.blit(GasStationImage,(50 + 40 * j, 50 + 40 * i))
                      # vẽ đường đi
                      pygame.draw.rect(screen, BLUE, [50 + 40 * j, 50 + 40 * i, 40, 40])

                    elif self.passable((i,j)):
                      # Không có vật cả thì hiển thị ô trống
                      pygame.draw.rect(screen, BLACK, [50 + 40 * j, 50 + 40 * i, 40, 40], 1)
                    else:
                      # Ô vật cản 
                      pygame.draw.rect(screen, GRAY, [50 + 40 * j, 50 + 40 * i, 40, 40])
        
            pygame.display.flip()

class SearchAlg:
  def __init__(self, grid, start, goal):
    self.grid = grid
    self.start = start
    self.goal = goal
    self.came_from = {}

  def trace_path(self):
    curr =  self.goal
    path = []
    while curr != self.start:
      path.append(curr)
      curr = self.came_from[curr]
    
    path.append(self.start)
    path.reverse()
    return path

  def heuristic(self,p1, p2, heu_type="Manhanttan"):
    if heu_type == "Manhanttan":
      return abs(p1[0]-p2[0]) + abs(p1[1]-p2[1])
    elif heu_type == "Euclidean":
      return numpy.linalg.norm(p1-p2)
    
    return sys.maxsize


  def a_star(self):
    n = int(Petrol[0]) #Số lít xăng đọc từ file
    petrol = n # N lít xăng
    open_list = PriorityQueue() #Khởi tạo danh sách
    gScore = {self.start: 0}  # lưu giá trị G của mỗi đỉnh
    fScore_start = self.heuristic(self.start, self.goal) # f = g + h = 0 + heu(start, goal)
    open_list.push(fScore_start, self.start) # push(value, label_node)
    self.came_from = {} # dùng để lưu dấu đường đi
    
    # List là rỗng và số lít xăng = 0 
    while (not open_list.is_empty()) and (petrol > 0):
        curr = open_list.pop()  # lấy đỉnh curr có fScore nhỏ nhất
        petrol -= 1 # Đi đường mới, và số lit xăng trừ đi 1
        # trả về (curr_fScore, curr_node)
        curr_fScore, curr_node = curr
        # Đích => kết thúc
        if curr_node == self.goal:
            print("Finded path!")
            path = self.trace_path() # vẽ đường
            self.grid.draw(path=path) #
            return True
        #Tìm xung quaynh node curr_node
        for next_node in self.grid.neighbors(curr_node):
            new_g = gScore[curr_node] + self.grid.A[next_node[0]][next_node[1]]  # next_g = curr_g + A[curr_node->next_node]
            if (next_node not in gScore) or (new_g < gScore[next_node]) and (next_node in self.grid.gas):

                # Tới cây xăng thì cập nhật lại xăng cho xe 
                if self.grid.A[next_node[0]][next_node[1]] == 0:
                  petrol = n
                # Giá trị của G mới
                gScore[next_node] = new_g
                fScore_next_node = gScore[next_node] + self.heuristic(next_node, self.goal) #f = g + h
                open_list.push(fScore_next_node, next_node)# cập nhật vào list
                self.came_from[next_node] = curr_node# lưu lại đường đi này

    print("Can not find path.")
    return False

  def BFS(self):
    queue = [] #Danh sách
    queue.append(self.start) # thêm phần tử bắt đầu
    visited = [] #lưu vị trí khi đi
    self.came_from = {}

    while len(queue) > 0:
      curr = queue.pop(0) #Lấy phần tử đầu tiên khi vào
      visited.append(curr) #Lưu lại vị trí đi

      #  goal => kết thúc
      if curr == self.goal:
        path = self.trace_path()
        self.grid.draw(path = path)
        return True

      for neighbor in self.grid.neighbors(curr):
        # Chưa từng đi qua
        if neighbor not in visited:
            queue.append(neighbor) #Cho vào hàng đợi
            visited.append(neighbor) #Lưu lại vị trí
            self.came_from[neighbor] = curr

    print("Can not find path.")
    return False    
  
  def DFS(self):
    stack = [] #Danh sách
    stack.append(self.start) #Thêm vào stack
    visited = [] #Danh sách lưu lại đường đã đi qua
    self.came_from = {}

    while len(stack) > 0:
      curr = stack.pop()  # Lấy ra khỏi ngăn xếp
      visited.append(curr) # Lưu lại đường đi

      # tìm thấy đích
      if curr == self.goal:
        path = self.trace_path()
        self.grid.draw(path = path)
        return True

      for neighbor in self.grid.neighbors(curr):
        # Chưa từng đi qua
        if neighbor not in visited:
            stack.append(neighbor) #Cho vào ngăn xếp
            visited.append(neighbor) #Lưu lại vị trí
            self.came_from[neighbor] = curr

    print("Can not find path.")
    return False    

  def UCS(self):
    n = int(Petrol[0]) #Số lít xăng đọc từ file
    open_list = PriorityQueue() #Danh sách
    petrol = n #Gán số lit xăng
    gScore = {self.start: 0}  # lưu giá trị G của mỗi đỉnh
    open_list.push(gScore, self.start) # push(value, label_node)
    self.came_from = {} # dùng để lưu dấu đường đi

    while (not open_list.is_empty()) and (petrol > 0):
        curr = open_list.pop()  # lấy đỉnh curr có fScore nhỏ nhất
        petrol -= 1
        print(curr)
        # trả về (curr_gScore, curr_node)
        curr_gScore, curr_node = curr
        if curr_node == self.goal:
            print("Finded path!")
            path = self.trace_path()
            self.grid.draw(path=path)
            return True
        #Tìm xung quaynh node curr_node
        for next_node in self.grid.neighbors(curr_node):
            new_g = gScore[curr_node] + self.grid.A[next_node[0]][next_node[1]]  # next_g = curr_g + A[curr_node->next_node]
            if (next_node not in gScore) or (new_g < gScore[next_node]):

                # Tới cây xăng thì cập nhật lại xăng cho xe 
                if self.grid.A[next_node[0]][next_node[1]] == 0:
                  petrol = n 
                
                gScore[next_node] = new_g
                # G của next_node
                gScore_next_node = gScore[next_node]
                open_list.push(gScore_next_node, next_node)# cập nhật vào list
                self.came_from[next_node] = curr_node #Lưu lại đường đã đi 

    print("Can not find path.")
    return False    


#Tạo mảng lưu giá trị vật cản đọc từ file
arraysWall = []
for array in Impediment:
     arraysWall.append((int(array[0]),int(array[1])))# lưu vào mảng

#Tạo mảng lưu giá trị cây xăng
arraysPetrol = []
for array in GasStation:
    arraysPetrol.append((int(array[0]),int(array[1])))#Lưu vào mảng


# Khởi tạo mảng A
rows, cols = (10, 10) 
#Mảng
A=[] 

# Mảng A với tất cả giá trị bằng 1
for i in range(cols): 
    col = [] 
    for j in range(rows):
        col.append(1) 
    A.append(col) 

# Tìm cây xăng và gán giá trị bằng 0
for i in range(len(A)):
  for j in range(len(A[i])):
    if (i,j) in arraysPetrol:
      A[i][j] = 0
      # Ô petrol (gas)
      screen.blit(GasStationImage,(50 + 40 * j, 50 + 40 * i))



g = Grid(A,arraysWall)# Vật cản và cay xăng trong mảng A
#g.draw()

# trạng thái bắt đầu( nơi bắt đầu) và kết thúc (đích)
start = (int(People[0][1]),int(People[0][0]))# bắt đầu
end = (int(Destination[0][1]),int(Destination[0][0]))# Kết thúc

search = SearchAlg(g, start, end)


search.a_star()
search.UCS()
search.DFS()
search.BFS()

#Đóng chương trình
pygame.quit()
