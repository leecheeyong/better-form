<template>
  <div id="app" class="min-h-screen bg-gray-50">
    <nav class="bg-white border-b border-gray-100 sticky top-0 z-40">
      <div class="max-w-6xl mx-auto px-6">
        <div class="flex justify-between items-center h-16">
          <div class="flex items-center space-x-3">
            <div
              class="w-8 h-8 bg-black rounded-lg flex items-center justify-center"
            >
              <span class="text-white font-bold text-sm">B</span>
            </div>
            <button
              @click="goHome"
              class="text-xl font-semibold text-gray-900 hover:text-gray-700 transition-colors"
            >
              Better Form
            </button>
          </div>

          <div
            v-if="!user && !isPublicForm"
            class="flex items-center space-x-3"
          >
            <button
              @click="
                showAuthModal = true;
                authMode = 'login';
              "
              class="text-gray-600 hover:text-gray-900 font-medium transition-colors"
            >
              Sign In
            </button>
            <button
              @click="
                showAuthModal = true;
                authMode = 'signup';
              "
              class="bg-black text-white px-4 py-2 rounded-lg font-medium hover:bg-gray-800 transition-colors"
            >
              Get Started
            </button>
          </div>

          <div v-else-if="user" class="flex items-center space-x-4">
            <div
              class="w-8 h-8 bg-gray-200 rounded-full flex items-center justify-center"
            >
              <span class="text-gray-600 font-medium text-sm">{{
                user.email?.[0]?.toUpperCase()
              }}</span>
            </div>
            <span class="text-sm text-gray-600">{{ user.email }}</span>
            <button
              @click="signOut"
              class="text-gray-500 hover:text-gray-700 transition-colors"
            >
              Sign Out
            </button>
          </div>
        </div>
      </div>
    </nav>

    <div
      v-if="showAuthModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-white rounded-2xl p-8 max-w-md w-full">
        <div class="text-center mb-8">
          <h2 class="text-2xl font-bold text-gray-900 mb-2">
            {{ authMode === "login" ? "Welcome back" : "Create account" }}
          </h2>
          <p class="text-gray-600">
            {{
              authMode === "login"
                ? "Sign in to your account"
                : "Start creating better forms today"
            }}
          </p>
        </div>

        <form @submit.prevent="handleAuth" class="space-y-4">
          <div>
            <input
              v-model="authForm.email"
              type="email"
              placeholder="Email address"
              required
              class="w-full px-4 py-3 border border-gray-200 rounded-xl focus:ring-2 focus:ring-black focus:border-transparent transition-all"
            />
          </div>
          <div>
            <input
              v-model="authForm.password"
              type="password"
              placeholder="Password (min 6 characters)"
              required
              minlength="6"
              class="w-full px-4 py-3 border border-gray-200 rounded-xl focus:ring-2 focus:ring-black focus:border-transparent transition-all"
            />
          </div>

          <button
            type="submit"
            :disabled="authLoading"
            class="w-full bg-black text-white py-3 rounded-xl font-medium hover:bg-gray-800 transition-colors disabled:opacity-50"
          >
            {{
              authLoading
                ? "Please wait..."
                : authMode === "login"
                  ? "Sign In"
                  : "Create Account"
            }}
          </button>
        </form>

        <div class="mt-6 text-center">
          <button
            @click="
              authMode = authMode === 'login' ? 'signup' : 'login';
              authError = '';
            "
            class="text-gray-600 hover:text-gray-900 transition-colors"
          >
            {{
              authMode === "login"
                ? "Don't have an account? Sign up"
                : "Already have an account? Sign in"
            }}
          </button>
        </div>

        <button
          @click="
            showAuthModal = false;
            authError = '';
          "
          class="absolute top-4 right-4 text-gray-400 hover:text-gray-600 transition-colors"
        >
          <X class="w-5 h-5" />
        </button>
      </div>
    </div>

    <main class="max-w-6xl mx-auto px-6 py-8">
      <div v-if="isPublicForm && publicForm">
        <div class="max-w-2xl mx-auto">
          <div
            class="bg-white rounded-3xl shadow-sm border border-gray-100 overflow-hidden"
          >
            <div class="h-1 bg-gray-100">
              <div
                class="h-full bg-black transition-all duration-500"
                :style="{
                  width: `${((currentQuestionIndex + 1) / Math.max(publicForm.questions.length, 1)) * 100}%`,
                }"
              ></div>
            </div>

            <div class="p-8 text-center border-b border-gray-100">
              <h1 class="text-2xl font-bold text-gray-900 mb-2">
                {{ publicForm.title }}
              </h1>
              <p v-if="publicForm.description" class="text-gray-600">
                {{ publicForm.description }}
              </p>
            </div>

            <div class="p-8 min-h-[400px] flex flex-col justify-center">
              <div
                v-if="currentQuestionIndex < publicForm.questions.length"
                class="space-y-8"
              >
                <div class="text-center">
                  <span class="text-sm text-gray-500 font-medium">
                    {{ currentQuestionIndex + 1 }} of
                    {{ publicForm.questions.length }}
                  </span>
                </div>

                <h2
                  class="text-2xl font-semibold text-gray-900 text-center leading-relaxed"
                >
                  {{ publicForm.questions[currentQuestionIndex].question }}
                  <span
                    v-if="publicForm.questions[currentQuestionIndex].required"
                    class="text-red-500"
                    >*</span
                  >
                </h2>

                <div
                  v-if="
                    ['text', 'email'].includes(
                      publicForm.questions[currentQuestionIndex].type,
                    )
                  "
                >
                  <input
                    v-model="publicFormResponses[currentQuestionIndex]"
                    :type="publicForm.questions[currentQuestionIndex].type"
                    placeholder="Type your answer..."
                    class="w-full px-6 py-4 border-2 border-gray-200 rounded-2xl focus:border-black focus:ring-0 transition-all text-center text-lg"
                    @keyup.enter="nextPublicQuestion"
                  />
                </div>

                <div
                  v-else-if="
                    publicForm.questions[currentQuestionIndex].type ===
                    'multiple'
                  "
                  class="space-y-3"
                >
                  <button
                    v-for="(option, optionIndex) in publicForm.questions[
                      currentQuestionIndex
                    ].options"
                    :key="optionIndex"
                    @click="
                      publicFormResponses[currentQuestionIndex] = option;
                      nextPublicQuestion();
                    "
                    :class="[
                      'w-full p-4 text-left border-2 rounded-2xl transition-all font-medium',
                      publicFormResponses[currentQuestionIndex] === option
                        ? 'border-black bg-gray-50 text-gray-900'
                        : 'border-gray-200 hover:border-gray-300 hover:bg-gray-50 text-gray-700',
                    ]"
                  >
                    {{ option }}
                  </button>
                </div>

                <div
                  v-else-if="
                    publicForm.questions[currentQuestionIndex].type === 'rating'
                  "
                >
                  <div class="flex justify-center space-x-2">
                    <button
                      v-for="rating in 5"
                      :key="rating"
                      @click="
                        publicFormResponses[currentQuestionIndex] = rating;
                        nextPublicQuestion();
                      "
                      :class="[
                        'p-3 transition-colors rounded-xl',
                        publicFormResponses[currentQuestionIndex] >= rating
                          ? 'text-yellow-400'
                          : 'text-gray-300 hover:text-yellow-300',
                      ]"
                    >
                      <Star class="w-8 h-8 fill-current" />
                    </button>
                  </div>
                </div>
              </div>

              <div v-else class="text-center space-y-6">
                <div
                  class="w-16 h-16 bg-green-100 rounded-2xl flex items-center justify-center mx-auto"
                >
                  <Check class="w-8 h-8 text-green-600" />
                </div>
                <h2 class="text-2xl font-bold text-gray-900">Thank you!</h2>
                <p class="text-gray-600">
                  Your response has been recorded successfully.
                </p>
                <button
                  @click="goHome"
                  class="bg-black text-white px-6 py-3 rounded-xl font-medium hover:bg-gray-800 transition-colors"
                >
                  Create Your Own Form
                </button>
              </div>
            </div>

            <div
              v-if="currentQuestionIndex < publicForm.questions.length"
              class="p-6 bg-gray-50 flex justify-between items-center"
            >
              <button
                v-if="currentQuestionIndex > 0"
                @click="previousPublicQuestion"
                class="flex items-center space-x-2 text-gray-600 hover:text-gray-900 transition-colors px-4 py-2 rounded-lg hover:bg-white"
              >
                <ChevronLeft class="w-4 h-4" />
                <span>Previous</span>
              </button>
              <div v-else></div>

              <button
                @click="nextPublicQuestion"
                :disabled="
                  isPublicQuestionRequired &&
                  !publicFormResponses[currentQuestionIndex]
                "
                :class="[
                  'flex items-center space-x-2 px-6 py-3 rounded-xl font-medium transition-all',
                  isPublicQuestionRequired &&
                  !publicFormResponses[currentQuestionIndex]
                    ? 'bg-gray-200 text-gray-400 cursor-not-allowed'
                    : 'bg-black text-white hover:bg-gray-800',
                ]"
              >
                <span>{{
                  currentQuestionIndex === publicForm.questions.length - 1
                    ? "Submit"
                    : "Next"
                }}</span>
                <ChevronRight class="w-4 h-4" />
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="isPublicForm && !publicForm && !loading">
        <div class="text-center py-20">
          <div
            class="w-16 h-16 bg-red-100 rounded-2xl flex items-center justify-center mx-auto mb-6"
          >
            <X class="w-8 h-8 text-red-600" />
          </div>
          <h2 class="text-2xl font-bold text-gray-900 mb-3">Form Not Found</h2>
          <p class="text-gray-600 mb-8">
            The form you're looking for doesn't exist or has been deleted.
          </p>
          <button
            @click="goHome"
            class="bg-black text-white px-6 py-3 rounded-xl font-medium hover:bg-gray-800 transition-colors"
          >
            Go Home
          </button>
        </div>
      </div>

      <div v-else-if="isPublicForm && loading">
        <div class="text-center py-20">
          <div
            class="animate-spin w-8 h-8 border-2 border-gray-300 border-t-black rounded-full mx-auto mb-4"
          ></div>
          <p class="text-gray-600">Loading form...</p>
        </div>
      </div>

      <div v-else-if="!user" class="text-center py-20">
        <h1 class="text-5xl font-bold text-gray-900 mb-6 leading-tight">
          Forms that people<br />
          <span class="text-gray-500">actually want to fill</span>
        </h1>
        <p
          class="text-xl text-gray-600 mb-12 max-w-2xl mx-auto leading-relaxed"
        >
          Create beautiful, interactive forms that feel more like conversations.
          Higher completion rates, better data quality.
        </p>

        <button
          @click="
            showAuthModal = true;
            authMode = 'signup';
          "
          class="bg-black text-white px-8 py-4 rounded-xl font-semibold text-lg hover:bg-gray-800 transition-all transform hover:scale-105 shadow-lg"
        >
          Start Building
        </button>

        <div class="grid md:grid-cols-3 gap-8 mt-24">
          <div class="text-center">
            <div
              class="w-12 h-12 bg-gray-100 rounded-2xl flex items-center justify-center mb-6 mx-auto"
            >
              <Zap class="w-6 h-6 text-gray-700" />
            </div>
            <h3 class="text-lg font-semibold mb-3 text-gray-900">
              One question at a time
            </h3>
            <p class="text-gray-600 leading-relaxed">
              Focus attention and increase completion rates with our slideshow
              format.
            </p>
          </div>
          <div class="text-center">
            <div
              class="w-12 h-12 bg-gray-100 rounded-2xl flex items-center justify-center mb-6 mx-auto"
            >
              <BarChart3 class="w-6 h-6 text-gray-700" />
            </div>
            <h3 class="text-lg font-semibold mb-3 text-gray-900">
              Real-time analytics
            </h3>
            <p class="text-gray-600 leading-relaxed">
              Track responses and analyze data in real-time with Firebase.
            </p>
          </div>
          <div class="text-center">
            <div
              class="w-12 h-12 bg-gray-100 rounded-2xl flex items-center justify-center mb-6 mx-auto"
            >
              <Share2 class="w-6 h-6 text-gray-700" />
            </div>
            <h3 class="text-lg font-semibold mb-3 text-gray-900">
              Share anywhere
            </h3>
            <p class="text-gray-600 leading-relaxed">
              One link works everywhere. No embedding, no complications.
            </p>
          </div>
        </div>
      </div>

      <div v-else-if="currentView === 'dashboard'">
        <div class="flex justify-between items-center mb-8">
          <div>
            <h1 class="text-3xl font-bold text-gray-900 mb-2">Your Forms</h1>
            <p class="text-gray-600">
              {{ forms.length }}
              {{ forms.length === 1 ? "form" : "forms" }} created
            </p>
          </div>
          <button
            @click="startNewForm"
            class="bg-black text-white px-6 py-3 rounded-xl font-medium hover:bg-gray-800 transition-colors flex items-center space-x-2"
          >
            <Plus class="w-5 h-5" />
            <span>New Form</span>
          </button>
        </div>

        <div v-if="loading" class="text-center py-20">
          <div
            class="animate-spin w-8 h-8 border-2 border-gray-300 border-t-black rounded-full mx-auto mb-4"
          ></div>
          <p class="text-gray-600">Loading your forms...</p>
        </div>

        <div v-else-if="forms.length === 0" class="text-center py-20">
          <div
            class="w-16 h-16 bg-gray-100 rounded-2xl flex items-center justify-center mx-auto mb-6"
          >
            <FileText class="w-8 h-8 text-gray-400" />
          </div>
          <h3 class="text-xl font-semibold text-gray-900 mb-3">
            Create your first form
          </h3>
          <p class="text-gray-600 mb-8 max-w-md mx-auto">
            Start building beautiful, interactive forms that your audience will
            love to complete.
          </p>
          <button
            @click="startNewForm"
            class="bg-black text-white px-6 py-3 rounded-xl font-medium hover:bg-gray-800 transition-colors"
          >
            Create Form
          </button>
        </div>

        <div v-else class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div
            v-for="form in forms"
            :key="form.id"
            class="bg-white rounded-2xl p-6 border border-gray-100 hover:border-gray-200 transition-all hover:shadow-sm group"
          >
            <div class="flex justify-between items-start mb-4">
              <h3 class="text-lg font-semibold text-gray-900 truncate pr-4">
                {{ form.title || "Untitled Form" }}
              </h3>
              <div
                class="flex space-x-1 opacity-0 group-hover:opacity-100 transition-opacity"
              >
                <button
                  @click="editForm(form)"
                  class="p-2 text-gray-400 hover:text-gray-600 transition-colors rounded-lg hover:bg-gray-50"
                >
                  <Edit2 class="w-4 h-4" />
                </button>
                <button
                  @click="viewResponses(form)"
                  class="p-2 text-gray-400 hover:text-blue-600 transition-colors rounded-lg hover:bg-gray-50"
                >
                  <BarChart3 class="w-4 h-4" />
                </button>
                <button
                  @click="deleteForm(form.id)"
                  class="p-2 text-gray-400 hover:text-red-600 transition-colors rounded-lg hover:bg-gray-50"
                >
                  <Trash2 class="w-4 h-4" />
                </button>
              </div>
            </div>

            <p class="text-gray-600 text-sm mb-4 line-clamp-2">
              {{ form.description || "No description" }}
            </p>

            <div
              class="text-xs text-gray-500 mb-4 font-mono bg-gray-50 px-2 py-1 rounded"
            >
              ID: {{ form.id }}
            </div>

            <div class="flex justify-between items-center text-sm mb-4">
              <span class="text-gray-500"
                >{{ form.questions?.length || 0 }} questions</span
              >
              <span class="text-green-600 font-medium"
                >{{ form.responseCount || 0 }} responses</span
              >
            </div>

            <div class="flex justify-between items-center text-sm">
              <div class="flex space-x-3">
                <button
                  @click="previewForm(form)"
                  class="text-gray-600 hover:text-gray-900 font-medium transition-colors"
                >
                  Preview
                </button>
                <button
                  @click="shareForm(form)"
                  class="text-black font-medium hover:text-gray-700 transition-colors"
                >
                  Share
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="currentView === 'builder'">
        <div class="flex justify-between items-center mb-8">
          <div class="flex items-center space-x-4">
            <button
              @click="currentView = 'dashboard'"
              class="p-2 text-gray-400 hover:text-gray-600 transition-colors rounded-lg hover:bg-gray-100"
            >
              <ArrowLeft class="w-5 h-5" />
            </button>
            <div>
              <h1 class="text-2xl font-bold text-gray-900">Form Builder</h1>
              <p class="text-gray-600">Design your form step by step</p>
              <p v-if="currentForm.id" class="text-xs text-gray-500 font-mono">
                ID: {{ currentForm.id }}
              </p>
            </div>
          </div>
          <div class="flex space-x-3">
            <button
              @click="previewCurrentForm"
              class="bg-gray-100 text-gray-700 px-4 py-2 rounded-lg font-medium hover:bg-gray-200 transition-colors"
            >
              Preview
            </button>
            <button
              @click="saveForm"
              :disabled="saving"
              class="bg-black text-white px-6 py-2 rounded-lg font-medium hover:bg-gray-800 transition-colors disabled:opacity-50"
            >
              {{ saving ? "Saving..." : "Save" }}
            </button>
          </div>
        </div>

        <div class="grid lg:grid-cols-4 gap-8">
          <div class="lg:col-span-1">
            <div
              class="bg-white rounded-2xl p-6 border border-gray-100 sticky top-24"
            >
              <h3 class="font-semibold mb-6 text-gray-900">Form Settings</h3>

              <div class="space-y-6">
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2"
                    >Title</label
                  >
                  <input
                    v-model="currentForm.title"
                    type="text"
                    placeholder="Form title"
                    class="w-full px-3 py-2 border border-gray-200 rounded-lg focus:ring-2 focus:ring-black focus:border-transparent transition-all"
                  />
                </div>

                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2"
                    >Description</label
                  >
                  <textarea
                    v-model="currentForm.description"
                    placeholder="Brief description"
                    rows="3"
                    class="w-full px-3 py-2 border border-gray-200 rounded-lg focus:ring-2 focus:ring-black focus:border-transparent transition-all resize-none"
                  ></textarea>
                </div>

                <div class="bg-blue-50 p-3 rounded-lg">
                  <div class="flex items-center space-x-2 mb-2">
                    <Database class="w-4 h-4 text-blue-600" />
                    <span class="text-sm font-medium text-blue-900"
                      >Firebase Storage</span
                    >
                  </div>
                  <p class="text-xs text-blue-700">
                    All responses are automatically stored in Firebase Firestore
                    with real-time analytics.
                  </p>
                </div>

                <div>
                  <h4 class="text-sm font-medium text-gray-700 mb-3">
                    Add Question
                  </h4>
                  <div class="space-y-2">
                    <button
                      @click="addQuestion('text')"
                      class="w-full p-3 border border-gray-200 rounded-lg hover:bg-gray-50 transition-colors text-left flex items-center space-x-3"
                    >
                      <Type class="w-4 h-4 text-gray-500" />
                      <span class="text-sm">Text Input</span>
                    </button>
                    <button
                      @click="addQuestion('multiple')"
                      class="w-full p-3 border border-gray-200 rounded-lg hover:bg-gray-50 transition-colors text-left flex items-center space-x-3"
                    >
                      <CheckSquare class="w-4 h-4 text-gray-500" />
                      <span class="text-sm">Multiple Choice</span>
                    </button>
                    <button
                      @click="addQuestion('email')"
                      class="w-full p-3 border border-gray-200 rounded-lg hover:bg-gray-50 transition-colors text-left flex items-center space-x-3"
                    >
                      <Mail class="w-4 h-4 text-gray-500" />
                      <span class="text-sm">Email</span>
                    </button>
                    <button
                      @click="addQuestion('rating')"
                      class="w-full p-3 border border-gray-200 rounded-lg hover:bg-gray-50 transition-colors text-left flex items-center space-x-3"
                    >
                      <Star class="w-4 h-4 text-gray-500" />
                      <span class="text-sm">Rating</span>
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="lg:col-span-3">
            <div
              v-if="currentForm.questions.length === 0"
              class="text-center py-20 bg-white rounded-2xl border border-gray-100"
            >
              <div
                class="w-12 h-12 bg-gray-100 rounded-2xl flex items-center justify-center mx-auto mb-4"
              >
                <HelpCircle class="w-6 h-6 text-gray-400" />
              </div>
              <h3 class="text-lg font-semibold text-gray-900 mb-2">
                Add your first question
              </h3>
              <p class="text-gray-600">
                Choose a question type from the sidebar to get started.
              </p>
            </div>

            <div v-else class="space-y-4">
              <div
                v-for="(question, index) in currentForm.questions"
                :key="question.id"
                class="bg-white rounded-2xl p-6 border border-gray-100"
              >
                <div class="flex justify-between items-start mb-6">
                  <div class="flex items-center space-x-3">
                    <div
                      class="w-8 h-8 bg-gray-100 text-gray-600 rounded-lg flex items-center justify-center text-sm font-medium"
                    >
                      {{ index + 1 }}
                    </div>
                    <span class="text-sm text-gray-500 capitalize"
                      >{{ question.type }} Question</span
                    >
                  </div>
                  <button
                    @click="removeQuestion(index)"
                    class="p-2 text-gray-400 hover:text-red-600 transition-colors rounded-lg hover:bg-gray-50"
                  >
                    <Trash2 class="w-4 h-4" />
                  </button>
                </div>

                <div class="space-y-4">
                  <div>
                    <input
                      v-model="question.question"
                      type="text"
                      placeholder="Enter your question"
                      class="w-full px-0 py-2 text-lg font-medium border-0 border-b-2 border-gray-200 focus:border-black focus:ring-0 transition-colors bg-transparent"
                    />
                  </div>

                  <div v-if="question.type === 'multiple'" class="space-y-3">
                    <div
                      v-for="(option, optionIndex) in question.options"
                      :key="optionIndex"
                      class="flex items-center space-x-3"
                    >
                      <div
                        class="w-4 h-4 border-2 border-gray-300 rounded-full"
                      ></div>
                      <input
                        v-model="question.options[optionIndex]"
                        type="text"
                        placeholder="Option text"
                        class="flex-1 px-3 py-2 border border-gray-200 rounded-lg focus:ring-2 focus:ring-black focus:border-transparent transition-all"
                      />
                      <button
                        @click="question.options.splice(optionIndex, 1)"
                        class="p-2 text-gray-400 hover:text-red-600 transition-colors"
                      >
                        <X class="w-4 h-4" />
                      </button>
                    </div>
                    <button
                      @click="question.options.push('')"
                      class="text-gray-600 hover:text-gray-900 text-sm font-medium transition-colors flex items-center space-x-2"
                    >
                      <Plus class="w-4 h-4" />
                      <span>Add option</span>
                    </button>
                  </div>

                  <div
                    class="flex items-center space-x-4 pt-4 border-t border-gray-100"
                  >
                    <label class="flex items-center space-x-2">
                      <input
                        v-model="question.required"
                        type="checkbox"
                        class="rounded border-gray-300 text-black focus:ring-black"
                      />
                      <span class="text-sm text-gray-700">Required</span>
                    </label>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="currentView === 'responses'">
        <div class="flex justify-between items-center mb-8">
          <div class="flex items-center space-x-4">
            <button
              @click="currentView = 'dashboard'"
              class="p-2 text-gray-400 hover:text-gray-600 transition-colors rounded-lg hover:bg-gray-100"
            >
              <ArrowLeft class="w-5 h-5" />
            </button>
            <div>
              <h1 class="text-2xl font-bold text-gray-900">Form Responses</h1>
              <p class="text-gray-600">
                {{ selectedForm?.title || "Form Analytics" }}
              </p>
            </div>
          </div>
          <div class="flex space-x-3">
            <button
              @click="exportResponses"
              class="bg-gray-100 text-gray-700 px-4 py-2 rounded-lg font-medium hover:bg-gray-200 transition-colors flex items-center space-x-2"
            >
              <Download class="w-4 h-4" />
              <span>Export CSV</span>
            </button>
          </div>
        </div>

        <div class="grid md:grid-cols-4 gap-6 mb-8">
          <div class="bg-white rounded-2xl p-6 border border-gray-100">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-sm text-gray-600">Total Responses</p>
                <p class="text-2xl font-bold text-gray-900">
                  {{ responses.length }}
                </p>
              </div>
              <div
                class="w-12 h-12 bg-blue-100 rounded-2xl flex items-center justify-center"
              >
                <BarChart3 class="w-6 h-6 text-blue-600" />
              </div>
            </div>
          </div>
          <div class="bg-white rounded-2xl p-6 border border-gray-100">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-sm text-gray-600">This Week</p>
                <p class="text-2xl font-bold text-gray-900">
                  {{ weeklyResponses }}
                </p>
              </div>
              <div
                class="w-12 h-12 bg-green-100 rounded-2xl flex items-center justify-center"
              >
                <TrendingUp class="w-6 h-6 text-green-600" />
              </div>
            </div>
          </div>
          <div class="bg-white rounded-2xl p-6 border border-gray-100">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-sm text-gray-600">Completion Rate</p>
                <p class="text-2xl font-bold text-gray-900">
                  {{ completionRate }}%
                </p>
              </div>
              <div
                class="w-12 h-12 bg-purple-100 rounded-2xl flex items-center justify-center"
              >
                <Target class="w-6 h-6 text-purple-600" />
              </div>
            </div>
          </div>
          <div class="bg-white rounded-2xl p-6 border border-gray-100">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-sm text-gray-600">Avg. Time</p>
                <p class="text-2xl font-bold text-gray-900">
                  {{ averageTime }}s
                </p>
              </div>
              <div
                class="w-12 h-12 bg-orange-100 rounded-2xl flex items-center justify-center"
              >
                <Clock class="w-6 h-6 text-orange-600" />
              </div>
            </div>
          </div>
        </div>

        <div
          class="bg-white rounded-2xl border border-gray-100 overflow-hidden"
        >
          <div class="p-6 border-b border-gray-100">
            <h3 class="text-lg font-semibold text-gray-900">
              Recent Responses
            </h3>
          </div>

          <div v-if="loading" class="p-8 text-center">
            <div
              class="animate-spin w-8 h-8 border-2 border-gray-300 border-t-black rounded-full mx-auto mb-4"
            ></div>
            <p class="text-gray-600">Loading responses...</p>
          </div>

          <div v-else-if="responses.length === 0" class="p-8 text-center">
            <div
              class="w-12 h-12 bg-gray-100 rounded-2xl flex items-center justify-center mx-auto mb-4"
            >
              <BarChart3 class="w-6 h-6 text-gray-400" />
            </div>
            <h3 class="text-lg font-semibold text-gray-900 mb-2">
              No responses yet
            </h3>
            <p class="text-gray-600">
              Share your form to start collecting responses.
            </p>
          </div>

          <div v-else class="overflow-x-auto">
            <table class="w-full">
              <thead class="bg-gray-50">
                <tr>
                  <th
                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                  >
                    Submitted
                  </th>
                  <th
                    v-for="question in selectedForm?.questions || []"
                    :key="question.id"
                    class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                  >
                    {{ question.question.substring(0, 30)
                    }}{{ question.question.length > 30 ? "..." : "" }}
                  </th>
                </tr>
              </thead>
              <tbody class="bg-white divide-y divide-gray-200">
                <tr
                  v-for="response in responses.slice(0, 50)"
                  :key="response.id"
                  class="hover:bg-gray-50"
                >
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                    {{ formatDate(response.submittedAt) }}
                  </td>
                  <td
                    v-for="answer in response.responses"
                    :key="answer.questionId"
                    class="px-6 py-4 text-sm text-gray-900"
                  >
                    <div class="max-w-xs truncate" :title="answer.answer">
                      {{ answer.answer || "-" }}
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <div v-else-if="currentView === 'preview'">
        <div class="flex justify-between items-center mb-8">
          <button
            @click="currentView = previousView"
            class="flex items-center space-x-2 text-gray-600 hover:text-gray-900 transition-colors"
          >
            <ArrowLeft class="w-5 h-5" />
            <span>Back</span>
          </button>
          <button
            @click="shareCurrentForm"
            class="bg-black text-white px-4 py-2 rounded-lg font-medium hover:bg-gray-800 transition-colors flex items-center space-x-2"
          >
            <Share2 class="w-4 h-4" />
            <span>Share</span>
          </button>
        </div>

        <div class="max-w-2xl mx-auto">
          <div
            class="bg-white rounded-3xl shadow-sm border border-gray-100 overflow-hidden"
          >
            <div class="h-1 bg-gray-100">
              <div
                class="h-full bg-black transition-all duration-500"
                :style="{
                  width: `${((currentQuestionIndex + 1) / Math.max(currentForm.questions.length, 1)) * 100}%`,
                }"
              ></div>
            </div>

            <div class="p-8 text-center border-b border-gray-100">
              <h1 class="text-2xl font-bold text-gray-900 mb-2">
                {{ currentForm.title || "Untitled Form" }}
              </h1>
              <p v-if="currentForm.description" class="text-gray-600">
                {{ currentForm.description }}
              </p>
            </div>

            <div class="p-8 min-h-[400px] flex flex-col justify-center">
              <div
                v-if="currentQuestionIndex < currentForm.questions.length"
                class="space-y-8"
              >
                <div class="text-center">
                  <span class="text-sm text-gray-500 font-medium">
                    {{ currentQuestionIndex + 1 }} of
                    {{ currentForm.questions.length }}
                  </span>
                </div>

                <h2
                  class="text-2xl font-semibold text-gray-900 text-center leading-relaxed"
                >
                  {{ currentForm.questions[currentQuestionIndex].question }}
                  <span
                    v-if="currentForm.questions[currentQuestionIndex].required"
                    class="text-red-500"
                    >*</span
                  >
                </h2>

                <div
                  v-if="
                    ['text', 'email'].includes(
                      currentForm.questions[currentQuestionIndex].type,
                    )
                  "
                >
                  <input
                    v-model="formResponses[currentQuestionIndex]"
                    :type="currentForm.questions[currentQuestionIndex].type"
                    placeholder="Type your answer..."
                    class="w-full px-6 py-4 border-2 border-gray-200 rounded-2xl focus:border-black focus:ring-0 transition-all text-center text-lg"
                  />
                </div>

                <div
                  v-else-if="
                    currentForm.questions[currentQuestionIndex].type ===
                    'multiple'
                  "
                  class="space-y-3"
                >
                  <button
                    v-for="(option, optionIndex) in currentForm.questions[
                      currentQuestionIndex
                    ].options"
                    :key="optionIndex"
                    @click="formResponses[currentQuestionIndex] = option"
                    :class="[
                      'w-full p-4 text-left border-2 rounded-2xl transition-all font-medium',
                      formResponses[currentQuestionIndex] === option
                        ? 'border-black bg-gray-50 text-gray-900'
                        : 'border-gray-200 hover:border-gray-300 hover:bg-gray-50 text-gray-700',
                    ]"
                  >
                    {{ option }}
                  </button>
                </div>

                <div
                  v-else-if="
                    currentForm.questions[currentQuestionIndex].type ===
                    'rating'
                  "
                >
                  <div class="flex justify-center space-x-2">
                    <button
                      v-for="rating in 5"
                      :key="rating"
                      @click="formResponses[currentQuestionIndex] = rating"
                      :class="[
                        'p-3 transition-colors rounded-xl',
                        formResponses[currentQuestionIndex] >= rating
                          ? 'text-yellow-400'
                          : 'text-gray-300 hover:text-yellow-300',
                      ]"
                    >
                      <Star class="w-8 h-8 fill-current" />
                    </button>
                  </div>
                </div>
              </div>

              <div v-else class="text-center space-y-6">
                <div
                  class="w-16 h-16 bg-green-100 rounded-2xl flex items-center justify-center mx-auto"
                >
                  <Check class="w-8 h-8 text-green-600" />
                </div>
                <h2 class="text-2xl font-bold text-gray-900">Thank you!</h2>
                <p class="text-gray-600">Your response has been recorded.</p>
              </div>
            </div>

            <div
              v-if="currentQuestionIndex < currentForm.questions.length"
              class="p-6 bg-gray-50 flex justify-between items-center"
            >
              <button
                v-if="currentQuestionIndex > 0"
                @click="previousQuestion"
                class="flex items-center space-x-2 text-gray-600 hover:text-gray-900 transition-colors px-4 py-2 rounded-lg hover:bg-white"
              >
                <ChevronLeft class="w-4 h-4" />
                <span>Previous</span>
              </button>
              <div v-else></div>

              <button
                @click="nextQuestion"
                :disabled="
                  isCurrentQuestionRequired &&
                  !formResponses[currentQuestionIndex]
                "
                :class="[
                  'flex items-center space-x-2 px-6 py-3 rounded-xl font-medium transition-all',
                  isCurrentQuestionRequired &&
                  !formResponses[currentQuestionIndex]
                    ? 'bg-gray-200 text-gray-400 cursor-not-allowed'
                    : 'bg-black text-white hover:bg-gray-800',
                ]"
              >
                <span>{{
                  currentQuestionIndex === currentForm.questions.length - 1
                    ? "Submit"
                    : "Next"
                }}</span>
                <ChevronRight class="w-4 h-4" />
              </button>
            </div>
          </div>
        </div>
      </div>
    </main>

    <div
      v-if="showShareModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-white rounded-2xl p-8 max-w-md w-full">
        <div class="flex justify-between items-center mb-6">
          <h3 class="text-xl font-semibold text-gray-900">Share Form</h3>
          <button
            @click="showShareModal = false"
            class="text-gray-400 hover:text-gray-600 transition-colors"
          >
            <X class="w-5 h-5" />
          </button>
        </div>

        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2"
              >Public Form Link</label
            >
            <div class="flex space-x-2">
              <input
                :value="shareUrl"
                readonly
                class="flex-1 px-4 py-3 border border-gray-200 rounded-xl bg-gray-50 text-gray-600 text-sm"
              />
              <button
                @click="copyShareUrl"
                class="bg-black text-white px-4 py-3 rounded-xl hover:bg-gray-800 transition-colors"
              >
                Copy
              </button>
            </div>
          </div>
          <div class="bg-gray-50 p-3 rounded-lg">
            <p class="text-xs text-gray-600 mb-1">
              Form ID:
              <span class="font-mono">{{ selectedFormForShare?.id }}</span>
            </p>
            <p class="text-xs text-gray-500">
              Anyone with this link can fill out your form. All responses are
              stored in Firebase.
            </p>
          </div>
        </div>
      </div>
    </div>

    <div
      v-if="showSubmissionSuccess"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-white rounded-2xl p-8 max-w-md w-full text-center">
        <div
          class="w-16 h-16 bg-green-100 rounded-2xl flex items-center justify-center mx-auto mb-4"
        >
          <Check class="w-8 h-8 text-green-600" />
        </div>
        <h3 class="text-xl font-semibold text-gray-900 mb-2">
          Response Submitted!
        </h3>
        <p class="text-gray-600 mb-6">
          Your form response has been successfully saved to Firebase.
        </p>
        <button
          @click="showSubmissionSuccess = false"
          class="bg-black text-white px-6 py-3 rounded-xl hover:bg-gray-800 transition-colors"
        >
          Close
        </button>
        <button
          v-if="user && currentForm.id && currentView === 'preview'"
          @click="() => { showSubmissionSuccess = false; viewResponses(currentForm); }"
          class="ml-4 bg-blue-600 text-white px-6 py-3 rounded-xl hover:bg-blue-700 transition-colors"
        >
          View Responses
        </button>
      </div>
    </div>

    <div
      v-if="globalAlert.message"
      class="fixed inset-x-0 bottom-4 z-50 mx-auto max-w-md w-full"
    >
      <div
        v-if="globalAlert.type === 'error'"
        class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-lg shadow-md flex items-center justify-between"
        role="alert"
      >
        <div>
          <p class="font-semibold">Error</p>
          <p class="text-sm">{{ globalAlert.message }}</p>
        </div>
        <button @click="globalAlert.message = ''" class="ml-4 text-red-400 hover:text-red-700">
          <X class="w-5 h-5" />
        </button>
      </div>
      <div
        v-else-if="globalAlert.type === 'success'"
        class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded-lg shadow-md flex items-center justify-between"
        role="alert"
      >
        <div>
          <p class="font-semibold">Success</p>
          <p class="text-sm">{{ globalAlert.message }}</p>
        </div>
        <button @click="globalAlert.message = ''" class="ml-4 text-green-400 hover:text-green-700">
          <X class="w-5 h-5" />
        </button>
      </div>
      <div
        v-else
        class="bg-blue-100 border border-blue-400 text-blue-700 px-4 py-3 rounded-lg shadow-md flex items-center justify-between"
        role="alert"
      >
        <div>
          <p class="font-semibold">Info</p>
          <p class="text-sm">{{ globalAlert.message }}</p>
        </div>
        <button @click="globalAlert.message = ''" class="ml-4 text-blue-400 hover:text-blue-700">
          <X class="w-5 h-5" />
        </button>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, reactive, computed, onMounted, watch } from "vue";
import {
  ArrowLeft,
  BarChart3,
  Check,
  CheckSquare,
  ChevronLeft,
  ChevronRight,
  Edit2,
  FileText,
  HelpCircle,
  Mail,
  Plus,
  Share2,
  Star,
  Trash2,
  Type,
  X,
  Zap,
  Database,
  Download,
  TrendingUp,
  Target,
  Clock,
} from "lucide-vue-next";

import { initializeApp } from "firebase/app";
import {
  getAuth,
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signOut as firebaseSignOut,
  onAuthStateChanged,
} from "firebase/auth";
import {
  getFirestore,
  collection,
  addDoc,
  getDocs,
  doc,
  deleteDoc,
  updateDoc,
  getDoc,
  query,
  where,
  orderBy,
  increment,
} from "firebase/firestore";

const firebaseConfig = {
  apiKey: import.meta.env.VITE_API_KEY,
  authDomain: "better-form-95050.firebaseapp.com",
  projectId: "better-form-95050",
  storageBucket: "better-form-95050.firebasestorage.app",
  messagingSenderId: "821126413132",
  appId: "1:821126413132:web:52814580a0ba8e3e3bade9",
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

const generateUniqueId = () => {
  return Date.now().toString(36) + Math.random().toString(36).substr(2, 9);
};

const user = ref(null);
const loading = ref(false);
const saving = ref(false);
const showAuthModal = ref(false);
const authMode = ref("login");
const authLoading = ref(false);
const authError = ref("");
const showSubmissionSuccess = ref(false);
const authForm = reactive({
  email: "",
  password: "",
});

const currentView = ref("dashboard");
const previousView = ref("dashboard");
const forms = ref([]);
const responses = ref([]);
const selectedForm = ref(null);
const currentForm = reactive({
  id: null,
  title: "",
  description: "",
  questions: [],
  createdAt: null,
  userId: null,
  responseCount: 0,
});

const currentQuestionIndex = ref(0);
const formResponses = ref({});
const showShareModal = ref(false);
const shareUrl = ref("");
const selectedFormForShare = ref(null);

const isPublicForm = ref(false);
const publicForm = ref(null);
const publicFormResponses = ref({});

const globalAlert = reactive({
  message: '',
  type: 'info', // 'info', 'error', 'success'
});

const weeklyResponses = computed(() => {
  const oneWeekAgo = new Date();
  oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
  return responses.value.filter((r) => {
    let date = r.submittedAt;
    if (date && typeof date.toDate === 'function') {
      date = date.toDate();
    }
    return date > oneWeekAgo;
  }).length;
});

const completionRate = computed(() => {
  if (responses.value.length === 0) return 0;
  const completed = responses.value.filter(
    (r) => r.responses && r.responses.length > 0,
  ).length;
  return Math.round((completed / responses.value.length) * 100);
});

const averageTime = computed(() => {
  if (responses.value.length === 0) return 0;
  const times = responses.value
    .filter((r) => r.completionTime)
    .map((r) => r.completionTime);
  if (times.length === 0) return 0;
  return Math.round(times.reduce((a, b) => a + b, 0) / times.length);
});

const isCurrentQuestionRequired = computed(() => {
  if (
    !currentForm.questions ||
    !currentForm.questions[currentQuestionIndex.value]
  ) {
    return false;
  }
  return currentForm.questions[currentQuestionIndex.value].required;
});

const isPublicQuestionRequired = computed(() => {
  if (
    !publicForm.value?.questions ||
    !publicForm.value.questions[currentQuestionIndex.value]
  ) {
    return false;
  }
  return publicForm.value.questions[currentQuestionIndex.value].required;
});

const checkForPublicForm = async () => {
  const urlParams = new URLSearchParams(window.location.search);
  const formId = urlParams.get("form");

  if (formId) {
    isPublicForm.value = true;
    await loadPublicForm(formId);
  }
};

const showAlert = (message, type = 'info', timeout = 5000) => {
  globalAlert.message = message;
  globalAlert.type = type;
  if (timeout > 0) {
    setTimeout(() => {
      if (globalAlert.message === message) globalAlert.message = '';
    }, timeout);
  }
};

const handleAuth = async () => {
  if (!authForm.email || !authForm.password) {
    showAlert('Please fill in all fields', 'error');
    return;
  }

  if (authForm.password.length < 6) {
    showAlert('Password must be at least 6 characters', 'error');
    return;
  }

  authLoading.value = true;
  authError.value = '';

  try {
    if (authMode.value === "signup") {
      const userCredential = await createUserWithEmailAndPassword(
        auth,
        authForm.email,
        authForm.password,
      );
      user.value = userCredential.user;
      showAlert('Account created successfully!', 'success');
    } else {
      const userCredential = await signInWithEmailAndPassword(
        auth,
        authForm.email,
        authForm.password,
      );
      user.value = userCredential.user;
      showAlert('Signed in successfully!', 'success');
    }

    showAuthModal.value = false;
    authForm.email = '';
    authForm.password = '';
    await loadForms();
  } catch (error) {
    let msg = '';
    switch (error.code) {
      case "auth/email-already-in-use":
        msg = "An account with this email already exists. Try signing in instead.";
        break;
      case "auth/weak-password":
        msg = "Password is too weak. Please choose a stronger password.";
        break;
      case "auth/invalid-email":
        msg = "Please enter a valid email address.";
        break;
      case "auth/user-not-found":
        msg = "No account found with this email. Try signing up instead.";
        break;
      case "auth/wrong-password":
        msg = "Incorrect password. Please try again.";
        break;
      case "auth/invalid-credential":
        msg = "Invalid email or password. Please check your credentials.";
        break;
      case "auth/too-many-requests":
        msg = "Too many failed attempts. Please try again later.";
        break;
      case "auth/network-request-failed":
        msg = "Network error. Please check your internet connection.";
        break;
      default:
        msg = error.message || "Authentication failed. Please try again.";
    }
    showAlert(msg, 'error');
  } finally {
    authLoading.value = false;
  }
};

const signOut = async () => {
  try {
    await firebaseSignOut(auth);
    user.value = null;
    forms.value = [];
    responses.value = [];
    currentView.value = "dashboard";
    isPublicForm.value = false;
    publicForm.value = null;
    showAlert('Signed out successfully!', 'success');
  } catch (error) {
    showAlert('Error signing out. Please try again.', 'error');
  }
};

const goHome = () => {
  isPublicForm.value = false;
  publicForm.value = null;
  currentView.value = user.value ? "dashboard" : "landing";
  window.history.replaceState({}, document.title, window.location.pathname);
};

const loadForms = async () => {
  if (!user.value) {
    console.log("No user authenticated, skipping form load");
    return;
  }

  loading.value = true;
  try {
    console.log("Loading forms for user:", user.value.uid);

    let q;
    try {
      q = query(
        collection(db, "forms"),
        where("userId", "==", user.value.uid),
        orderBy("createdAt", "desc"),
      );
    } catch (indexError) {
      console.log("Index not available, using simple query");
      q = query(collection(db, "forms"), where("userId", "==", user.value.uid));
    }

    const querySnapshot = await getDocs(q);
    forms.value = querySnapshot.docs.map((doc) => ({
      id: doc.id,
      ...doc.data(),
      createdAt: doc.data().createdAt?.toDate() || new Date(),
      updatedAt: doc.data().updatedAt?.toDate() || new Date(),
    }));

    forms.value.sort((a, b) => b.createdAt - a.createdAt);

    console.log(
      `Successfully loaded ${forms.value.length} forms for user ${user.value.email}`,
    );
  } catch (error) {
    console.error("Error loading forms:", error);
    console.error("Error code:", error.code);
    console.error("Error message:", error.message);

    if (
      error.code === "failed-precondition" ||
      error.message.includes("index")
    ) {
      alert(
        "Database index required. Creating the index will take a few minutes. You can continue using the app, but forms may not be sorted by date.",
      );
      console.log("Index creation link should appear in console above");
    } else if (error.code === "permission-denied") {
      alert("Permission denied. Please check your Firestore security rules.");
    } else {
      alert(`Error loading forms: ${error.message}`);
    }
  } finally {
    loading.value = false;
  }
};

const loadPublicForm = async (formId) => {
  loading.value = true;
  try {
    const docRef = doc(db, "forms", formId);
    const docSnap = await getDoc(docRef);

    if (docSnap.exists()) {
      const formData = docSnap.data();
      let createdAt = formData.createdAt;
      let updatedAt = formData.updatedAt;
      if (createdAt && typeof createdAt.toDate === 'function') createdAt = createdAt.toDate();
      if (updatedAt && typeof updatedAt.toDate === 'function') updatedAt = updatedAt.toDate();
      publicForm.value = {
        id: docSnap.id,
        ...formData,
        createdAt: createdAt || new Date(),
        updatedAt: updatedAt || new Date(),
      };
      currentQuestionIndex.value = 0;
      publicFormResponses.value = {};
      console.log("Public form loaded:", publicForm.value.title);
    } else {
      publicForm.value = null;
      console.log("Public form not found:", formId);
    }
  } catch (error) {
    console.error("Error loading public form:", error);
    publicForm.value = null;
  } finally {
    loading.value = false;
  }
};

const loadResponses = async (formId) => {
  loading.value = true;
  try {
    const q = query(
      collection(db, "responses"),
      where("formId", "==", formId),
      orderBy("submittedAt", "desc"),
    );
    const querySnapshot = await getDocs(q);
    responses.value = querySnapshot.docs.map((doc) => ({
      id: doc.id,
      ...doc.data(),
      submittedAt: doc.data().submittedAt?.toDate() || new Date(),
    }));

    console.log(
      `Loaded ${responses.value.length} responses for form ${formId}`,
    );
  } catch (error) {
    console.error("Error loading responses:", error);
    alert("Error loading responses. Please try again.");
  } finally {
    loading.value = false;
  }
};

const startNewForm = () => {
  resetCurrentForm();
  currentView.value = "builder";
};

const addQuestion = (type) => {
  const question = {
    id: "q" + generateUniqueId(),
    type,
    question: "",
    required: false,
  };

  if (type === "multiple") {
    question.options = ["Option 1", "Option 2"];
  }

  currentForm.questions.push(question);
};

const removeQuestion = (index) => {
  currentForm.questions.splice(index, 1);
};

const saveForm = async () => {
  if (!currentForm.title.trim()) {
    showAlert('Please enter a form title', 'error');
    return;
  }

  if (currentForm.questions.length === 0) {
    showAlert('Please add at least one question', 'error');
    return;
  }

  if (!user.value) {
    showAlert('You must be logged in to save forms', 'error');
    return;
  }

  saving.value = true;

  try {
    const formData = {
      title: currentForm.title,
      description: currentForm.description,
      questions: currentForm.questions,
      userId: user.value.uid,
      createdAt: currentForm.createdAt || new Date(),
      updatedAt: new Date(),
      responseCount: currentForm.responseCount || 0,
    };

    if (currentForm.id) {
      await updateDoc(doc(db, "forms", currentForm.id), formData);
      const index = forms.value.findIndex((f) => f.id === currentForm.id);
      if (index !== -1) {
        forms.value[index] = { id: currentForm.id, ...formData };
      }
    } else {
      const docRef = await addDoc(collection(db, "forms"), formData);
      const newForm = { id: docRef.id, ...formData };
      forms.value.unshift(newForm);
      currentForm.id = docRef.id;
    }

    showAlert('Form saved successfully!', 'success');
    currentView.value = "dashboard";
    resetCurrentForm();
  } catch (error) {
    if (error.code === "permission-denied") {
      showAlert('Permission denied. Please check your Firestore security rules and ensure you are properly authenticated.', 'error', 8000);
    } else {
      showAlert(`Error saving form: ${error.message}`, 'error', 8000);
    }
  } finally {
    saving.value = false;
  }
};

const editForm = (form) => {
  Object.assign(currentForm, JSON.parse(JSON.stringify(form)));
  currentView.value = "builder";
};

const deleteForm = async (formId) => {
  if (!confirm("Delete this form? This action cannot be undone.")) return;

  try {
    await deleteDoc(doc(db, "forms", formId));
    forms.value = forms.value.filter((f) => f.id !== formId);
    showAlert('Form deleted.', 'success');
  } catch (error) {
    showAlert('Error deleting form. Please try again.', 'error');
  }
};

const viewResponses = async (form) => {
  selectedForm.value = form;
  await loadResponses(form.id);
  currentView.value = "responses";
};

const previewForm = (form) => {
  Object.assign(currentForm, JSON.parse(JSON.stringify(form)));
  currentQuestionIndex.value = 0;
  formResponses.value = {};
  previousView.value = "dashboard";
  currentView.value = "preview";
};

const previewCurrentForm = () => {
  if (currentForm.questions.length === 0) {
    alert("Add at least one question to preview");
    return;
  }

  currentQuestionIndex.value = 0;
  formResponses.value = {};
  previousView.value = "builder";
  currentView.value = "preview";
};

const shareForm = (form) => {
  selectedFormForShare.value = form;
  shareUrl.value = `${window.location.origin}?form=${form.id}`;
  showShareModal.value = true;
};

const shareCurrentForm = () => {
  if (!currentForm.id) {
    alert("Please save the form first to get a shareable link");
    return;
  }
  selectedFormForShare.value = currentForm;
  shareUrl.value = `${window.location.origin}?form=${currentForm.id}`;
  showShareModal.value = true;
};

const copyShareUrl = async () => {
  try {
    await navigator.clipboard.writeText(shareUrl.value);
    showAlert('Link copied to clipboard!', 'success');
  } catch (err) {
    const textArea = document.createElement("textarea");
    textArea.value = shareUrl.value;
    document.body.appendChild(textArea);
    textArea.select();
    document.execCommand("copy");
    document.body.removeChild(textArea);
    showAlert('Link copied!', 'success');
  }
};

const exportResponses = () => {
  if (responses.value.length === 0) {
    showAlert('No responses to export', 'error');
    return;
  }

  const headers = [
    "Submitted At",
    ...selectedForm.value.questions.map((q) => q.question),
  ];
  const csvContent = [
    headers.join(","),
    ...responses.value.map((response) => {
      const row = [
        formatDate(response.submittedAt),
        ...response.responses.map((r) => `"${r.answer || ""}"`),
      ];
      return row.join(",");
    }),
  ].join("\n");

  const blob = new Blob([csvContent], { type: "text/csv" });
  const url = window.URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = `${selectedForm.value.title || "form"}-responses.csv`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  window.URL.revokeObjectURL(url);
  showAlert('Responses exported as CSV.', 'success');
};

const formatDate = (date) => {
  if (!date) return "-";
  return new Intl.DateTimeFormat("en-US", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  }).format(date);
};

const nextQuestion = async () => {
  if (currentQuestionIndex.value < currentForm.questions.length - 1) {
    currentQuestionIndex.value++;
  } else {
    await submitForm();
    currentQuestionIndex.value++;
  }
};

const previousQuestion = () => {
  if (currentQuestionIndex.value > 0) {
    currentQuestionIndex.value--;
  }
};

const nextPublicQuestion = async () => {
  if (currentQuestionIndex.value < publicForm.value.questions.length - 1) {
    currentQuestionIndex.value++;
  } else {
    await submitPublicForm();
    currentQuestionIndex.value++;
  }
};

const previousPublicQuestion = () => {
  if (currentQuestionIndex.value > 0) {
    currentQuestionIndex.value--;
  }
};

const submitForm = async () => {
  const responses = currentForm.questions.map((question, index) => ({
    questionId: question.id,
    question: question.question,
    answer: formResponses.value[index] || "",
    type: question.type,
  }));

  console.log("Form submitted (preview):", responses);
};

const submitPublicForm = async () => {
  try {
    const startTime = Date.now();
    const responses = publicForm.value.questions.map((question, index) => ({
      questionId: question.id,
      question: question.question,
      answer: publicFormResponses.value[index] || "",
      type: question.type,
    }));

    const responseData = {
      formId: publicForm.value.id,
      responses,
      submittedAt: new Date(),
      userAgent: navigator.userAgent,
      completionTime: Math.round((Date.now() - startTime) / 1000),
      responseId: "resp_" + generateUniqueId(),
    };

    await addDoc(collection(db, "responses"), responseData);
    await updateDoc(doc(db, "forms", publicForm.value.id), {
      responseCount: increment(1),
      lastResponseAt: new Date(),
    });

    showSubmissionSuccess.value = true;
  } catch (error) {
    showAlert('Error submitting form. Please try again.', 'error');
  }
};

const resetCurrentForm = () => {
  Object.assign(currentForm, {
    id: null,
    title: "",
    description: "",
    questions: [],
    createdAt: null,
    userId: null,
    responseCount: 0,
  });
};

onMounted(async () => {
  await checkForPublicForm();

  onAuthStateChanged(auth, async (firebaseUser) => {
    if (firebaseUser) {
      user.value = firebaseUser;
      console.log("User authenticated:", firebaseUser.email);
      if (!isPublicForm.value) {
        await loadForms();
      }
    } else {
      user.value = null;
      forms.value = [];
      responses.value = [];
      console.log("User not authenticated");
    }
  });
});
</script>
<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>

