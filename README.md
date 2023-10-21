# Petmatch
{
  "users": {
    "user1": {
      "password": "password1",
      "liked_dogs": [],
      "disliked_dogs": []
    },
    "user2": {
      "password": "password2",
      "liked_dogs": [],
      "disliked_dogs": []
    }
  }
}
from kivy.app import App
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.filechooser import FileChooserListView
from kivy.uix.popup import Popup
from kivy.uix.image import Image

class PetMatchApp(App):
    def build(self):
        self.user_profiles = {
            "user1": {"password": "password1", "liked_dogs": [], "disliked_dogs": []},
            "user2": {"password": "password2", "liked_dogs": [], "disliked_dogs": []}
        }
        self.current_user = None
        self.current_dog = None
        self.dog_profiles = []  # Lista de perfis de cães
        self.matches = []  # Lista de combinações entre usuários

        self.screen_manager = ScreenManager()

        self.login_screen = LoginScreen(self)
        self.main_screen = MainScreen(self)
        self.user_profile_screen = UserProfileScreen(self)
        self.edit_dog_profile_screen = EditDogProfileScreen(self)
        self.create_dog_profile_screen = CreateDogProfileScreen(self)
        self.rejected_profiles_screen = RejectedProfilesScreen(self)
        self.search_breed_screen = SearchBreedScreen(self)
        self.matched_profiles_screen = MatchedProfilesScreen(self)

        self.screen_manager.add_widget(self.login_screen)
        self.screen_manager.add_widget(self.main_screen)
        self.screen_manager.add_widget(self.user_profile_screen)
        self.screen_manager.add_widget(self.edit_dog_profile_screen)
        self.screen_manager.add_widget(self.create_dog_profile_screen)
        self.screen_manager.add_widget(self.rejected_profiles_screen)
        self.screen_manager.add_widget(self.search_breed_screen)
        self.screen_manager.add_widget(self.matched_profiles_screen)

        return self.screen_manager

class LoginScreen(Screen):
    def __init__(self, app, **kwargs):
        super(LoginScreen, self).__init__(**kwargs)
        self.app = app

        layout = BoxLayout(orientation='vertical')
        self.username_input = TextInput(hint_text='Username')
        self.password_input = TextInput(hint_text='Password', password=True)
        login_button = Button(text='Login', on_press=self.login)
        register_button = Button(text='Register', on_press=self.register)
        layout.add_widget(self.username_input)
        layout.add_widget(self.password_input)
        layout.add_widget(login_button)
        layout.add_widget(register_button)
        self.add_widget(layout)

    def login(self, instance):
        username = self.username_input.text
        password = self.password_input.text

        if username in self.app.user_profiles and self.app.user_profiles[username]["password"] == password:
            self.app.current_user = username
            self.app.screen_manager.current = 'main'
        else:
            print("Login failed. Check your credentials.")

    def register(self, instance):
        username = self.username_input.text
        password = self.password_input.text

        if username not in self.app.user_profiles:
            self.app.user_profiles[username] = {"password": password, "liked_dogs": [], "disliked_dogs": []}
            self.app.current_user = username
            self.app.screen_manager.current = 'main'
        else:
            print("Username already exists. Try a different one.")

class MainScreen(Screen):
    def __init__(self, app, **kwargs):
        super(MainScreen, self).__init__(**kwargs)
        self.app = app
        self.current_profile_index = 0

        layout = BoxLayout(orientation='vertical')

        self.dog_images = ImageCarousel()
        self.profile_info = Label(text='', size_hint=(None, None), size=(400, 150))
        like_button = Button(text='Curtir', size_hint=(None, None), size=(100, 50))
        dislike_button = Button(text='Rejeitar', size_hint=(None, None), size=(100, 50))
        view_profile_button = Button(text='Perfil', on_press=self.view_user_profile, size_hint=(None, None), size=(100, 50))
        previous_button = Button(text='Anterior', on_press=self.show_previous_profile, size_hint=(None, None), size=(100, 50))
        next_button = Button(text='Próximo', on_press=self.show_next_profile, size_hint=(None, None), size=(100, 50))
        search_button = Button(text='Pesquisar por Raça', on_press=self.search_by_breed, size_hint=(None, None), size=(150, 50))
        rejected_profiles_button = Button(text='Perfis Rejeitados', on_press=self.view_rejected_profiles, size_hint=(None, None), size=(150, 50))
        create_dog_profile_button = Button(text='Criar Perfil de Cão', on_press=self.create_dog_profile, size_hint=(None, None), size=(150, 50))
        view_matches_button = Button(text='Ver Combinações', on_press=self.view_matches, size_hint=(None, None), size=(150, 50))

        like_button.bind(on_press=self.like_pet)
        dislike_button.bind(on_press=self.dislike_pet)

        layout.add_widget(self.dog_images)
        layout.add_widget(self.profile_info)
        button_layout = BoxLayout(orientation='horizontal', spacing=10)
        button_layout.add_widget(like_button)
        button_layout.add_widget(dislike_button)
        button_layout.add_widget(view_profile_button)
        button_layout.add_widget(previous_button)
        button_layout.add_widget(next_button)
        button_layout.add_widget(search_button)
        button_layout.add_widget(rejected_profiles_button)
        button_layout.add_widget(create_dog_profile_button)
        button_layout.add_widget(view_matches_button)
        layout.add_widget(button_layout)

        self.update_dog_profile()

        self.add_widget(layout)

    def update_dog_profile(self):
        if self.current_profile_index < len(self.app.dog_profiles):
            profile = self.app.dog_profiles[self.current_profile_index]
            self.dog_images.set_images(profile["images"])
            self.profile_info.text = f"Nome: {profile['name']}\nRaça: {profile['breed']}\nIdade: {profile['age']}\nDescrição: {profile['description']}"
            self.app.current_dog = profile
        else:
            self.dog_images.clear_images()
            self.profile_info.text = "Não há mais perfis de cães para exibir."
            self.app.current_dog = None

    def like_pet(self, instance):
        if self.current_profile_index < len(self.app.dog_profiles):
            liked_dog = self.app.dog_profiles[self.current_profile_index]
            self.app.user_profiles[self.app.current_user]["liked_dogs"].append(liked_dog)
            self.current_profile_index += 1
            self.update_dog_profile()

    def dislike_pet(self, instance):
        if self.current_profile_index < len(self.app.dog_profiles):
            self.app.user_profiles[self.app.current_user]["disliked_dogs"].append(self.app.dog_profiles[self.current_profile_index])
            self.current_profile_index += 1
            self.update_dog_profile()

    def show_previous_profile(self, instance):
        if self.current_profile_index > 0:
            self.current_profile_index -= 1
            self.update_dog_profile()

    def show_next_profile(self, instance):
        if self.current_profile_index < len(self.app.dog_profiles) - 1:
            self.current_profile_index += 1
            self.update_dog_profile()

    def view_user_profile(self, instance):
        self.app.user_profile_screen.load_user_info()
        self.app.screen_manager.current = 'user_profile'

    def search_by_breed(self, instance):
        self.app.search_breed_screen.load_breeds()
        self.app.screen_manager.current = 'search_breed'

    def view_rejected_profiles(self, instance):
        self.app.rejected_profiles_screen.load_rejected_profiles()
        self.app.screen_manager.current = 'rejected_profiles'

    def create_dog_profile(self, instance):
        self.app.screen_manager.current = 'create_dog_profile'

    def view_matches(self, instance):
        self.app.matched_profiles_screen.load_matches()
        self.app.screen_manager.current = 'matched_profiles'

class UserProfileScreen(Screen):
    def __init__(self, app, **kwargs):
        super(UserProfileScreen, self).__init__(**kwargs)
        self.app = app
        self.user_info_label = Label(text='', size_hint_y=None, height=100)
        self.dogs_info_label = Label(text='', size_hint_y=None, height=300)
        back_button = Button(text='Voltar', on_press=self.go_back)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(self.user_info_label)
        layout.add_widget(self.dogs_info_label)
        layout.add_widget(back_button)

        self.add_widget(layout)

    def load_user_info(self):
        if self.app.current_user in self.app.user_profiles:
            user_info = self.app.user_profiles[self.app.current_user]
            liked_dogs = user_info["liked_dogs"]
            user_info_text = f"Usuário: {self.app.current_user}"
            liked_dogs_text = f"Cães Curtidos: {[dog['name'] for dog in liked_dogs]}"
            self.user_info_label.text = user_info_text
            self.dogs_info_label.text = liked_dogs_text

    def go_back(self, instance):
        self.app.screen_manager.current = 'main'

class EditDogProfileScreen(Screen):
    def __init__(self, app, **kwargs):
        super(EditDogProfileScreen, self).__init__(**kwargs)
        self.app = app
        self.dog_name_input = TextInput(hint_text='Nome do Cão')
        self.dog_breed_input = TextInput(hint_text='Raça do Cão')
        self.dog_age_input = TextInput(hint_text='Idade do Cão')
        self.description_input = TextInput(hint_text='Descrição do Cão')
        self.upload_image_button = Button(text='Carregar Imagem', on_press=self.upload_image)
        save_button = Button(text='Salvar', on_press=self.save_dog_profile)
        back_button = Button(text='Voltar', on_press=self.go_back)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(self.dog_name_input)
        layout.add_widget(self.dog_breed_input)
        layout.add_widget(self.dog_age_input)
        layout.add_widget(self.description_input)
        layout.add_widget(self.upload_image_button)
        button_layout = BoxLayout(orientation='horizontal')
        button_layout.add_widget(save_button)
        button_layout.add_widget(back_button)
        layout.add_widget(button_layout)

        self.add_widget(layout)

    def upload_image(self, instance):
        file_chooser = FileChooserListView()
        file_chooser.bind(on_submit=self.select_image)
        popup = Popup(title='Escolha uma imagem', content=file_chooser, size_hint=(None, None), size=(600, 400))
        popup.open()

    def select_image(self, instance, selection, *args):
        if selection:
            selected_image_path = selection[0]
            self.app.current_dog["images"].append(selected_image_path)
            self.app.screen_manager.current = 'edit_dog_profile'

    def save_dog_profile(self, instance):
        if self.app.current_dog:
            self.app.current_dog["name"] = self.dog_name_input.text
            self.app.current_dog["breed"] = self.dog_breed_input.text
            self.app.current_dog["age"] = self.dog_age_input.text
            self.app.current_dog["description"] = self.description_input.text
            self.app.screen_manager.current = 'user_profile'

    def go_back(self, instance):
        self.app.screen_manager.current = 'user_profile'

class CreateDogProfileScreen(Screen):
    def __init__(self, app, **kwargs):
        super(CreateDogProfileScreen, self).__init__(**kwargs)
        self.app = app
        self.dog_name_input = TextInput(hint_text='Nome do Cão')
        self.dog_breed_input = TextInput(hint_text='Raça do Cão')
        self.dog_age_input = TextInput(hint_text='Idade do Cão')
        self.description_input = TextInput(hint_text='Descrição do Cão')
        self.upload_image_button = Button(text='Carregar Imagem', on_press=self.upload_image)
        create_button = Button(text='Criar Perfil de Cão', on_press=self.create_dog_profile)
        back_button = Button(text='Voltar', on_press=self.go_back)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(self.dog_name_input)
        layout.add_widget(self.dog_breed_input)
        layout.add_widget(self.dog_age_input)
        layout.add_widget(self.description_input)
        layout.add_widget(self.upload_image_button)
        button_layout = BoxLayout(orientation='horizontal')
        button_layout.add_widget(create_button)
        button_layout.add_widget(back_button)
        layout.add_widget(button_layout)

        self.add_widget(layout)

    def upload_image(self, instance):
        file_chooser = FileChooserListView()
        file_chooser.bind(on_submit=self.select_image)
        popup = Popup(title='Escolha uma imagem', content=file_chooser, size_hint=(None, None), size=(600, 400))
        popup.open()

    def select_image(self, instance, selection, *args):
        if selection:
            selected_image_path = selection[0]
            self.app.current_dog["images"].append(selected_image_path)
            self.app.screen_manager.current = 'create_dog_profile'

    def create_dog_profile(self, instance):
        dog_name = self.dog_name_input.text
        dog_breed = self.dog_breed_input.text
        dog_age = self.dog_age_input.text
        dog_description = self.description_input.text

        if dog_name and dog_breed and dog_age and dog_description:
            new_dog_profile = {
                "name": dog_name,
                "breed": dog_breed,
                "age": dog_age,
                "description": dog_description,
                "images": ["path_to_default_image.png"]
            }
            self.app.dog_profiles.append(new_dog_profile)
            self.app.screen_manager.current = 'main'

    def go_back(self, instance):
        self.app.screen_manager.current = 'main'

class RejectedProfilesScreen(Screen):
    def __init__(self, app, **kwargs):
        super(RejectedProfilesScreen, self).__init__(**kwargs)
        self.app = app
        self.rejected_profiles_label = Label(text='', size_hint_y=None, height=300)
        back_button = Button(text='Voltar', on_press=self.go_back)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(self.rejected_profiles_label)
        layout.add_widget(back_button)

        self.add_widget(layout)

    def load_rejected_profiles(self):
        if self.app.current_user in self.app.user_profiles:
            rejected_dogs = self.app.user_profiles[self.app.current_user]["disliked_dogs"]
            rejected_profiles_text = "Perfis Rejeitados:\n"
            for dog in rejected_dogs:
                rejected_profiles_text += f"Nome: {dog['name']}, Raça: {dog['breed']}, Idade: {dog['age']}\n"
            self.rejected_profiles_label.text = rejected_profiles_text

    def go_back(self, instance):
        self.app.screen_manager.current = 'main'

class SearchBreedScreen(Screen):
    def __init__(self, app, **kwargs):
        super(SearchBreedScreen, self).__init__(**kwargs)
        self.app = app
        self.breed_input = TextInput(hint_text='Raça do Cão')
        search_button = Button(text='Pesquisar', on_press=self.search_dogs)
        back_button = Button(text='Voltar', on_press=self.go_back)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(self.breed_input)
        button_layout = BoxLayout(orientation='horizontal')
        button_layout.add_widget(search_button)
        button_layout.add_widget(back_button)
        layout.add_widget(button_layout)

        self.result_label = Label(text='', size_hint_y=None, height=300)
        layout.add_widget(self.result_label)

        self.add_widget(layout)

    def search_dogs(self, instance):
        breed_to search = self.breed_input.text
        matching_dogs = [dog for dog in self.app.dog_profiles if dog['breed'].lower() == breed_to_search.lower()]
        result_text = "Cães com a raça especificada:\n"
        if matching_dogs:
            for dog in matching_dogs:
                result_text += f"Nome: {dog['name']}, Raça: {dog['breed']}, Idade: {dog['age']}\n"
        else:
            result_text = "Nenhum cão encontrado com a raça especificada."

        self.result_label.text = result_text

    def go_back(self, instance):
        self.result_label.text = ''
        self.app.screen_manager.current = 'main'

class MatchedProfilesScreen(Screen):
    def __init__(self, app, **kwargs):
        super(MatchedProfilesScreen, self).__init__(**kwargs)
        self.app = app
        self.matched_profiles_label = Label(text='', size_hint_y=None, height=300)
        back_button = Button(text='Voltar', on_press=self.go_back)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(self.matched_profiles_label)
        layout.add_widget(back_button)

        self.add_widget(layout)

    def load_matches(self):
        matches = [match for match in self.app.matches if self.app.current_user in match["users"]]
        matched_profiles_text = "Perfis Correspondentes:\n"
        for match in matches:
            users = match["users"]
            matched_dog = match["dog"]
            matched_profiles_text += f"Usuários: {users[0]} e {users[1]}\n"
            matched_profiles_text += f"Cão Correspondente: {matched_dog['name']}, Raça: {matched_dog['breed']}, Idade: {matched_dog['age']}\n\n"
        self.matched_profiles_label.text = matched_profiles_text

    def go_back(self, instance):
        self.app.screen_manager.current = 'main'

class ImageCarousel(BoxLayout):
    def __init__(self, **kwargs):
        super(ImageCarousel, self).__init__(**kwargs)
        self.orientation = 'horizontal'
        self.images = []

    def set_images(self, image_paths):
        self.clear_images()
        for path in image_paths:
            image = Image(source=path, size_hint=(None, None), size=(200, 200))
            self.add_widget(image)
            self.images.append(image)

    def clear_images(self):
        for image in self.images:
            self.remove_widget(image)
        self.images = []

if __name__ == '__main__':
    PetMatchApp().run()

