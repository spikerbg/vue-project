<template>
  <div class="min-h-screen bg-gray-50">
    <!-- Header -->
    <header class="border-b bg-white">
      <div class="container mx-auto px-4 py-4">
        <nav class="flex items-center justify-between">
          <div class="flex items-center space-x-2">
            <ShoppingBagIcon class="h-6 w-6" />
            <span class="text-xl font-semibold">Clay Shop</span>
          </div>

          <div class="hidden md:flex space-x-8">
            <a href="#" class="text-gray-700 hover:text-gray-900">Home</a>
            <a href="#" class="text-gray-700 hover:text-gray-900">Shop</a>
            <a href="#" class="text-gray-700 hover:text-gray-900">Blog</a>
            <a href="#" class="text-gray-700 hover:text-gray-900">Contact</a>
          </div>

          <div class="flex items-center space-x-4">
            <button @click="toggleCart" class="relative">
              <ShoppingCartIcon class="h-6 w-6" />
              <span
                v-if="cartItems.length"
                class="absolute -top-2 -right-2 bg-pink-500 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center"
              >
                {{ cartItems.length }}
              </span>
            </button>
          </div>
        </nav>
      </div>
    </header>

    <!-- Продуктова страница -->
    <main class="container mx-auto px-4 py-8">
      <div class="grid md:grid-cols-2 gap-8">
        <!-- Галерия -->
        <div class="space-y-4">
          <div class="aspect-square bg-white rounded-lg overflow-hidden">
            <img :src="selectedImage" :alt="product.name" class="w-full h-full object-cover" />
          </div>
          <div class="grid grid-cols-5 gap-2">
            <button
              v-for="(image, index) in product.images"
              :key="index"
              @click="selectedImage = image"
              class="aspect-square bg-white rounded-lg overflow-hidden border-2 relative"
              :class="{ 'border-pink-500': selectedImage === image }"
            >
              <img :src="image" :alt="product.name" class="w-full h-full object-cover" />
              <!-- Добавям градиент на последната снимка -->
              <div
                v-if="index === product.images.length - 1"
                class="absolute inset-0 bg-gradient-to-l from-white to-transparent"
              ></div>
            </button>
          </div>
        </div>

        <!-- Продуктова информация -->

        <div class="space-y-6">
          <div class="flex justify-between">
            <div class="inline-block px-3 py-1 bg-pink-100 text-sm rounded-full">Popular</div>
            <div class="inline-block px-3 py-1 bg-pink-100 text-sm rounded-full">
              <Heart color="#ff00ff" />
            </div>
          </div>
          <h1 class="text-3xl font-semibold">{{ product.name }}</h1>

          <div class="flex items-center space-x-2">
            <div class="flex">
              <StarIcon
                v-for="i in 5"
                :key="i"
                :class="i <= averageRating ? 'text-yellow-400' : 'text-gray-300'"
                class="h-5 w-5"
              />
            </div>
            <span class="text-sm text-gray-600"
              >Rating: {{ averageRating }} based on {{ comments.length }} reviews</span
            >
          </div>

          <div class="flex space-x-8 border-t border-b py-4">
            <button
              v-for="tab in ['Info', 'Brand', 'Delivery']"
              :key="tab"
              @click="activeTab = tab"
              class="text-sm font-medium"
              :class="activeTab === tab ? 'text-pink-500 underline' : 'text-gray-500'"
            >
              {{ tab }}
            </button>
          </div>

          <p class="text-gray-600">{{ product.description }}</p>

          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium mb-2">Size</label>
              <div class="flex space-x-2">
                <button
                  v-for="size in product.sizes"
                  :key="size"
                  @click="selectedSize = size"
                  class="px-4 py-2 rounded-md"
                  :class="selectedSize === size ? 'bg-pink-500 text-white' : 'bg-gray-100'"
                >
                  {{ size }}
                </button>
              </div>
            </div>

            <div>
              <label class="block text-sm font-medium mb-2">Color</label>
              <div class="flex space-x-2">
                <button
                  v-for="color in product.colors"
                  :key="color"
                  @click="selectedColor = color"
                  class="w-8 h-8 rounded-full border-2"
                  :class="selectedColor === color ? 'border-pink-500' : 'border-transparent'"
                  :style="{ backgroundColor: color }"
                />
              </div>
            </div>
          </div>

          <div class="flex items-center space-x-4">
            <span class="text-2xl font-bold">${{ product.price }}</span>
            <button
              @click="addToCart"
              class="px-6 py-3 bg-pink-500 text-white rounded-md hover:bg-pink-600 transition-colors"
            >
              Add to cart
            </button>
          </div>
          <div>
            <!-- Рейтингова система -->
            <div class="mt-4">
              <div class="flex">
                <StarIcon
                  v-for="i in 5"
                  :key="i"
                  :class="i <= rating ? 'text-yellow-400' : 'text-gray-300'"
                  class="h-5 w-5 cursor-pointer"
                  @click="setRating(i)"
                />
              </div>
            </div>
            <p class="mt-2 text-gray-600">Add your comment here</p>

            <!-- Добавяне на коментар -->
            <textarea
              v-model="comment"
              placeholder="Leave a comment..."
              class="w-full mt-4 p-3 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              rows="4"
            ></textarea>
            <button
              @click="submitComment"
              :disabled="rating === 0 || !comment.trim()"
              class="mt-2 px-4 py-2 bg-pink-500 text-white font-semibold rounded-md disabled:bg-gray-400 disabled:cursor-not-allowed"
            >
              Submit
            </button>

            <!-- Показване на всички коментари -->
            <div v-if="comments.length > 0" class="mt-6">
              <h3 class="text-xl font-semibold text-gray-800">Comments:</h3>
              <div
                v-for="(comment, index) in comments"
                :key="index"
                class="mt-4 p-4 border border-gray-300 rounded-lg"
              >
                <div class="flex items-center space-x-2">
                  <StarIcon
                    v-for="i in 5"
                    :key="i"
                    :class="i <= comment.rating ? 'text-yellow-400' : 'text-gray-300'"
                    class="h-5 w-5"
                  />
                </div>
                <p class="mt-2 text-gray-600">{{ comment.text }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Popup Количка -->
    <Transition name="slide">
      <div v-if="isCartOpen" class="fixed inset-0 z-50">
        <div class="absolute inset-0 bg-black bg-opacity-50" @click="toggleCart" />
        <div class="absolute right-0 top-0 h-full w-full max-w-md bg-white">
          <div class="p-6">
            <div class="flex items-center justify-between mb-8">
              <div class="flex space-x-8">
                <button
                  v-for="step in ['Cart', 'Checkout', 'Shipping', 'Done']"
                  :key="step"
                  class="text-sm font-medium"
                  :class="currentStep === step ? 'text-pink-500' : 'text-gray-400'"
                >
                  {{ step }}
                </button>
              </div>
              <button @click="toggleCart">
                <XIcon class="h-6 w-6" />
              </button>
            </div>

            <div class="space-y-6">
              <!-- Проверяваме дали количката е празна -->
              <p v-if="cartItems.length === 0">Cart is empty</p>

              <!-- Показване на продукти в количка -->
              <div v-for="item in cartItems" :key="item.id" class="flex space-x-4" v-else>
                <img :src="item.image" :alt="item.name" class="w-24 h-24 object-cover rounded-lg" />
                <div class="flex-1">
                  <h3 class="font-medium">{{ item.name }}</h3>
                  <div class="text-sm text-gray-500 mt-1">
                    <div>Size: {{ item.size }}</div>
                    <div class="flex items-center">
                      Color:
                      <span
                        class="w-4 h-4 rounded-full ml-1"
                        :style="{ backgroundColor: item.color }"
                      />
                    </div>
                  </div>
                </div>
                <div class="flex flex-col items-end">
                  <span class="font-medium">${{ item.price }}</span>
                  <div class="flex items-center space-x-2 mt-2">
                    <button
                      @click="updateQuantity(item, -1)"
                      class="w-8 h-8 flex items-center justify-center rounded-full bg-gray-100"
                    >
                      <MinusIcon class="h-4 w-4" />
                    </button>
                    <span>{{ item.quantity }}</span>
                    <button
                      @click="updateQuantity(item, 1)"
                      class="w-8 h-8 flex items-center justify-center rounded-full bg-gray-100"
                    >
                      <PlusIcon class="h-4 w-4" />
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div class="mt-8 pt-8 border-t">
              <div class="flex justify-between text-lg font-medium">
                <span>Total amount</span>
                <span>${{ totalAmount }}</span>
              </div>
              <div class="mt-8 grid grid-cols-2 gap-4">
                <button
                  class="px-6 py-3 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50"
                >
                  To shop
                </button>
                <button class="px-6 py-3 bg-pink-500 text-white rounded-md hover:bg-pink-600">
                  Continue
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import {
  ShoppingBagIcon,
  ShoppingCartIcon,
  StarIcon,
  XIcon,
  MinusIcon,
  PlusIcon,
  Heart,
} from 'lucide-vue-next'

// Product Data
const product = ref({
  name: 'Black Valentino dress with tulle',
  description:
    'Dress with tulle and collar Peter Pan from RED/Valentino (Black Valentino). Peter Pan collar, tulle panels, sleeveless model, concealed back zipper and pleated skirt. Black colour.',
  price: 1315,
  rating: 4,
  reviews: 132,
  sizes: ['XS', 'S', 'M'],
  colors: ['#000000', '#4A90E2', '#50E3C2'],
  images: ['/image1.jpg', '/image2.jpg', '/image2.webp', '/image1.jpg', '/image1.jpg'],
})
// Управление на стейт за коментари и рейтинг
const rating = ref(0)
const comment = ref('')
const comments = ref([])
// Функцията на рейтинга
const setRating = (star) => {
  rating.value = star
}
const submitComment = () => {
  if (rating.value > 0 && comment.value.trim()) {
    const newComment = {
      rating: rating.value,
      text: comment.value.trim(),
    }
    comments.value.push(newComment)

    // Чистим инпутите
    rating.value = 0
    comment.value = ''
  }
}
const averageRating = computed(() => {
  const total = comments.value.reduce((sum, c) => sum + c.rating, 0)
  return comments.value.length > 0 ? (total / comments.value.length).toFixed(1) : 0
})

// Стейт на продукта
const selectedImage = ref(product.value.images[0])
const selectedSize = ref(product.value.sizes[0])
const selectedColor = ref(product.value.colors[0])
const activeTab = ref('Info')

// Стейт на количката
const isCartOpen = ref(false)
const currentStep = ref('Cart')
const cartItems = ref([])

// Зареждаме локал сториджа
onMounted(() => {
  const savedCart = localStorage.getItem('cart')
  if (savedCart) {
    cartItems.value = JSON.parse(savedCart)
  }
})

// Запазваме локалсториджа
const saveCart = () => {
  localStorage.setItem('cart', JSON.stringify(cartItems.value))
}

// Функция на количката
const toggleCart = () => {
  isCartOpen.value = !isCartOpen.value
}

const addToCart = () => {
  const item = {
    id: Date.now(),
    name: product.value.name,
    price: product.value.price,
    size: selectedSize.value,
    color: selectedColor.value,
    image: selectedImage.value,
    quantity: 1,
  }
  cartItems.value.push(item)
  saveCart()
  isCartOpen.value = true
}

const updateQuantity = (item, change) => {
  const index = cartItems.value.findIndex((i) => i.id === item.id)
  if (index !== -1) {
    const newQuantity = item.quantity + change
    if (newQuantity > 0) {
      cartItems.value[index].quantity = newQuantity
    } else {
      cartItems.value.splice(index, 1)
    }
    saveCart()
  }
}

// Изчисление
const totalAmount = computed(() => {
  return cartItems.value.reduce((total, item) => total + item.price * item.quantity, 0)
})
</script>

<style scoped>
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease-in-out;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}
</style>
