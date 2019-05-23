import vk_api
from vk_api.longpoll import VkLongPoll, VkEventType
from vk_api.keyboard import VkKeyboard, VkKeyboardColor
from vk_api.utils import get_random_id
import json
import random



GROUP_TOKEN = '8fe022337ed514128263df1c1ded158a244ecc275fdeec78a05d692d46dd41a208a874e8f042902dd8815'
APP_TOKEN = 'de398a3cde398a3cde398a3c4bde501794dde39de398a3c82fba0525358a06f961f1566'


class user(object):
    """
        class of VK user

    """

    def __init__(self, vk_user_object, previous_point, depth):
        self.id_of_user       = vk_user_object['id']
        self.name             = vk_user_object['first_name']
        self.surname          = vk_user_object['last_name']
        self.previous_point   = previous_point
        self.depth            = depth
        self.finnish          = False
    def get_route(self, path):
        ''' returns descending sequences of user objects down to the client_user object, excluding it '''
        path.append(self)
        if self.previous_point.previous_point == -1:
            return path
        return self.previous_point.get_route(path=path)

    def __str__(self):
        return '{}\nФамилия: {}\nИмя: {}\nhttps://www.vk.com/id{}'.format(self.depth, self.surname, self.name, self.id_of_user)


def send_message(text, id_of_user):
    if not text:
        text = "Empty message"
    vk.messages.send( #Отправляем сообщение
        user_id=id_of_user,
        message=text,
        random_id = random.randint(1000000000,100000000000)
        )



def offer(usr,user_id):
    res=False
    send_message('Это тот, кто нужен?\nhttps://vk.com/id{}\nИмя: {}\nФамилия: {}'.format(usr.id_of_user, usr.name, usr.surname),user_id)
    keyboard = VkKeyboard(one_time=True)
    keyboard.add_button('Да', color=VkKeyboardColor.POSITIVE, payload='0')
    keyboard.add_button('Нет', color=VkKeyboardColor.DEFAULT, payload='1')
    vk.messages.send(
                peer_id=user_id,
                random_id=random.randint(1000000000,100000000000),
                keyboard=keyboard.get_keyboard(),
                message='Выберите пользователя'
                )
    for event in longpoll.listen():
        if event.type == VkEventType.MESSAGE_NEW and event.to_me and event.text:
            if event.text == 'Да':
                print('Yes')
                #raise KeybooardInterrupt
                return True
            else:
                print('No')
                return False

    #TODO: need send_message function defined --- done
    #TODO: show buttons 'Yes' or 'No'
    #TODO: res = ...
    '''True or False'''




def get_branch(current_user, id_of_user ):
    global target
    global visited_ids
    #static maxi_matches
    branch=[]
    #response = vk.method('friends.get?fields="nickname"&user_id=' + str(current_user.id_of_user) + '&lang=0')

    response = vk_app.method('friends.get?fields="nickname"&user_id={}&lang=0'.format(current_user.id_of_user))
    response = response['items']
    for i in response:
        if (i['id'] != current_user.id_of_user) and (not(i['id'] in visited_ids)):
            buff=user(i, current_user, current_user.depth+1)
            visited_ids.append(i['id'])
            #print(buff.name+'|'+target['name'][0]+' '+buff.surname+'|'+target['surname'][0])
            if buff.name in target['name']  and buff.surname in target['surname']:
                if offer(buff,id_of_user):
                    buff.finnish = 'v'
                    return [buff]
            branch.append(buff)
    return branch





def bfs_search_path(start_user):
    queue = [start_user]
    global visited_ids
    visited_ids = []
    while queue:
        vertex = queue.pop(0)
        visited_ids.append(vertex.id_of_user)
        try:
            data = get_branch(vertex,start_user.id_of_user)

            for next in data:
                if next.finnish == 'v' :
                    print("HOORAY\n")
                    return [str(i)   for i in next.get_route([])]
                else:
                    if next.depth == 6:
                        return ['Ой.']
                    queue.append(next)
        except KeyboardInterrupt:
            raise
        except Exception as e:
            print(e)
            try:
                print("{} {}".format(vertex.surname, vertex.name))
            except:
                print("")

############################################################
############################################################
############################################################

def main():

    global vk
    global vk_app
    vk_session = vk_api.VkApi(token=GROUP_TOKEN)
    vk = vk_session.get_api()
    vk_app = vk_api.VkApi(token=APP_TOKEN)
    global longpoll
    longpoll = VkLongPoll(vk_session)
    global target
    target = {}
    global visited_ids
    visited_ids = []
    #TODO: need user data
    #TODO: need longPoll wrapping code
    flag = True
    for event in longpoll.listen():
        if event.type == VkEventType.MESSAGE_NEW and event.to_me and event.text:
                try:
                    if flag:
                        print(1)
                        target.update({ 'name':event.text.split()})
                        send_message("Теперь введи фамилию", event.user_id)

                        flag = not flag

                    else:
                        print(2)
                        target.update({ 'surname':event.text.split()})
                        send_message("Начинаю обработку", event.user_id)
                        flag = not flag
                except:
                    send_message('Снова.', event.user_id)

                while flag:
                    print(3)
                    if 'Дмитрий' in target['name'] and 'Стародубцев' in target['surname']:
                        send_message('Возможно, вы искали "Солнышко"?',event.user_id)
                        flag = not flag
                        break
                    #resp = vk_app.method('users.get?fields="nickname"&user_ids={}&lang=0'.format(event.user_id) )['items'][0]

                    #seed_user_first_name =  resp['first_name']
                    #seed_user_last_name =  resp['last_name']
                    #del resp
                    start_user = user({'id': event.user_id,
                                       'first_name':'ivan',
                                       'last_name':'ivanon'},-1,0)
                    print(target)
                    tmp_h = bfs_search_path(start_user)[::-1]
                    try:
                        for i in tmp_h:
                            send_message(i,event.user_id)
                        #flag = not flag
                        break
                    except:
                        print(tmp_h)
                        raise

                    flag = True











if __name__ == '__main__':
    main()
