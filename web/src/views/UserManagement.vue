<script setup lang="ts">
import type { UserCard } from '@/stores/user'
import { onMounted, ref } from 'vue'
import api from '@/api'
import BaseButton from '@/components/ui/BaseButton.vue'
import BaseInput from '@/components/ui/BaseInput.vue'
import { useToastStore } from '@/stores/toast'
import { useUserStore } from '@/stores/user'

interface UserWithPassword {
  username: string
  password: string
  role: string
  card: UserCard | null
  accountLimit: number
}

interface EditForm {
  username: string
  password: string
  accountLimit: number
  expiresAt: string
  isPermanent: boolean
}

const userStore = useUserStore()
const toast = useToastStore()

const users = ref<UserWithPassword[]>([])
const loading = ref(false)
const showEditModal = ref(false)
const selectedUser = ref<UserWithPassword | null>(null)
const editForm = ref<EditForm>({
  username: '',
  password: '',
  accountLimit: 2,
  expiresAt: '',
  isPermanent: false,
})
const editLoading = ref(false)

async function fetchUsers() {
  loading.value = true
  try {
    const result = await userStore.getAllUsersWithPassword()
    if (result.ok) {
      users.value = result.data
    }
    else {
      toast.error(result.error || '获取用户列表失败')
    }
  }
  catch (e: any) {
    toast.error(e.message || '获取用户列表失败')
  }
  finally {
    loading.value = false
  }
}

async function toggleUserStatus(user: UserWithPassword) {
  try {
    const updates: Partial<UserCard> = { enabled: !user.card?.enabled }
    const result = await userStore.updateUser(user.username, updates)
    if (result.ok) {
      toast.success(user.card?.enabled ? '用户已封禁' : '用户已解封')
      await fetchUsers()
    }
    else {
      toast.error(result.error || '操作失败')
    }
  }
  catch (e: any) {
    toast.error(e.message || '操作失败')
  }
}

async function deleteUser(user: UserWithPassword) {
  if (!confirm(`确定要删除用户 ${user.username} 吗？此操作不可恢复！`))
    return

  try {
    const result = await userStore.deleteUser(user.username)
    if (result.ok) {
      toast.success('用户删除成功')
      await fetchUsers()
    }
    else {
      toast.error(result.error || '删除用户失败')
    }
  }
  catch (e: any) {
    toast.error(e.message || '删除用户失败')
  }
}

function openEditModal(user: UserWithPassword) {
  selectedUser.value = user
  editForm.value = {
    username: user.username,
    password: user.password,
    accountLimit: user.accountLimit || 2,
    expiresAt: user.card?.expiresAt ? formatDateTimeLocal(user.card.expiresAt) : '',
    isPermanent: user.card?.days === -1,
  }
  showEditModal.value = true
}

function formatDateTimeLocal(timestamp: number): string {
  const date = new Date(timestamp)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  return `${year}-${month}-${day}T${hours}:${minutes}`
}

async function handleEdit() {
  if (!selectedUser.value)
    return

  if (!editForm.value.username.trim()) {
    toast.warning('用户名不能为空')
    return
  }

  if (!editForm.value.password.trim()) {
    toast.warning('密码不能为空')
    return
  }

  editLoading.value = true
  try {
    const expiresAtValue = editForm.value.isPermanent
      ? null
      : (editForm.value.expiresAt ? new Date(editForm.value.expiresAt).getTime() : null)

    const res = await api.post(`/api/admin/users/${selectedUser.value.username}/edit`, {
      newUsername: editForm.value.username,
      password: editForm.value.password,
      accountLimit: editForm.value.accountLimit,
      expiresAt: expiresAtValue,
      isPermanent: editForm.value.isPermanent,
    })

    if (res.data.ok) {
      toast.success('用户信息已更新')
      showEditModal.value = false
      await fetchUsers()
    }
    else {
      toast.error(res.data.error || '更新失败')
    }
  }
  catch (e: any) {
    toast.error(e?.response?.data?.error || e?.message || '更新失败')
  }
  finally {
    editLoading.value = false
  }
}

async function copyPassword(password: string) {
  try {
    if (navigator.clipboard && navigator.clipboard.writeText) {
      await navigator.clipboard.writeText(password)
      toast.success('密码已复制到剪贴板')
    }
    else {
      const textArea = document.createElement('textarea')
      textArea.value = password
      textArea.style.position = 'fixed'
      textArea.style.opacity = '0'
      document.body.appendChild(textArea)
      textArea.select()
      document.execCommand('copy')
      toast.success('密码已复制到剪贴板')
      document.body.removeChild(textArea)
    }
  }
  catch (e) {
    toast.error('复制失败，请手动复制')
    console.error('复制失败:', e)
  }
}

function formatDate(timestamp: number | null) {
  if (!timestamp)
    return '永久有效'
  return new Date(timestamp).toLocaleString('zh-CN')
}

function getDaysLabel(days: number) {
  if (days === -1)
    return '永久'
  return `${days}天`
}

function isExpired(card: UserCard | null) {
  if (!card?.expiresAt)
    return false
  return Date.now() > card.expiresAt
}

onMounted(() => {
  fetchUsers()
})
</script>

<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <h1 class="text-2xl text-gray-900 font-bold dark:text-white">
        用户管理
      </h1>
      <BaseButton variant="primary" @click="fetchUsers">
        刷新
      </BaseButton>
    </div>

    <!-- 用户列表 -->
    <div class="overflow-hidden rounded-lg bg-white shadow dark:bg-gray-800">
      <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
          <thead class="bg-gray-50 dark:bg-gray-700">
            <tr>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                用户名
              </th>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                密码
              </th>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                角色
              </th>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                额度
              </th>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                时长
              </th>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                过期时间
              </th>
              <th class="px-6 py-3 text-left text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                状态
              </th>
              <th class="px-6 py-3 text-right text-xs text-gray-500 font-medium tracking-wider uppercase dark:text-gray-300">
                操作
              </th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200 dark:bg-gray-800 dark:divide-gray-700">
            <tr v-for="user in users" :key="user.username">
              <td class="whitespace-nowrap px-6 py-4 text-sm text-gray-900 font-medium dark:text-white">
                {{ user.username }}
              </td>
              <td class="whitespace-nowrap px-6 py-4 text-sm text-gray-900 dark:text-white">
                <div class="flex items-center space-x-2">
                  <span class="font-mono">{{ user.password }}</span>
                  <button
                    class="text-sm text-blue-600 dark:text-blue-400 hover:text-blue-500"
                    @click="copyPassword(user.password)"
                  >
                    复制
                  </button>
                </div>
              </td>
              <td class="whitespace-nowrap px-6 py-4 text-sm text-gray-900 dark:text-white">
                <span
                  class="inline-flex rounded-full px-2 text-xs font-semibold leading-5"
                  :class="user.role === 'admin' ? 'bg-purple-100 text-purple-800 dark:bg-purple-900 dark:text-purple-200' : 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-200'"
                >
                  {{ user.role === 'admin' ? '管理员' : '用户' }}
                </span>
              </td>
              <td class="whitespace-nowrap px-6 py-4 text-sm text-gray-900 dark:text-white">
                <span
                  class="inline-flex rounded-full px-2 text-xs font-semibold leading-5"
                  :class="user.role === 'admin' ? 'bg-purple-100 text-purple-800 dark:bg-purple-900 dark:text-purple-200' : 'bg-orange-100 text-orange-800 dark:bg-orange-900 dark:text-orange-200'"
                >
                  {{ user.role === 'admin' ? '无限制' : `${user.accountLimit || 2}个` }}
                </span>
              </td>
              <td class="whitespace-nowrap px-6 py-4 text-sm text-gray-900 dark:text-white">
                {{ user.card ? getDaysLabel(user.card.days) : '无' }}
              </td>
              <td class="whitespace-nowrap px-6 py-4 text-sm" :class="isExpired(user.card) ? 'text-red-600 dark:text-red-400' : 'text-gray-900 dark:text-white'">
                {{ formatDate(user.card?.expiresAt || null) }}
              </td>
              <td class="whitespace-nowrap px-6 py-4">
                <span
                  v-if="user.card"
                  class="inline-flex rounded-full px-2 text-xs font-semibold leading-5"
                  :class="user.card.enabled === false ? 'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200' : (isExpired(user.card) ? 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200' : 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200')"
                >
                  {{ user.card.enabled === false ? '封禁' : (isExpired(user.card) ? '已过期' : '正常') }}
                </span>
                <span v-else class="text-gray-500 dark:text-gray-400">-</span>
              </td>
              <td class="whitespace-nowrap px-6 py-4 text-right text-sm font-medium">
                <button
                  v-if="user.role !== 'admin'"
                  class="mr-3 text-blue-600 dark:text-blue-400 hover:text-blue-900 dark:hover:text-blue-300"
                  @click="openEditModal(user)"
                >
                  编辑
                </button>
                <button
                  v-if="user.role !== 'admin' && user.card"
                  class="mr-3 text-yellow-600 dark:text-yellow-400 hover:text-yellow-900 dark:hover:text-yellow-300"
                  @click="toggleUserStatus(user)"
                >
                  {{ user.card.enabled === false ? '解封' : '封禁' }}
                </button>
                <button
                  v-if="user.role !== 'admin'"
                  class="text-red-600 dark:text-red-400 hover:text-red-900 dark:hover:text-red-300"
                  @click="deleteUser(user)"
                >
                  删除
                </button>
              </td>
            </tr>
            <tr v-if="users.length === 0">
              <td colspan="8" class="px-6 py-4 text-center text-gray-500 dark:text-gray-400">
                暂无用户
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- 编辑弹窗 -->
    <div
      v-if="showEditModal"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50"
      @click.self="showEditModal = false"
    >
      <div class="max-w-md w-full rounded-lg bg-white p-6 dark:bg-gray-800">
        <h2 class="mb-4 text-xl text-gray-900 font-bold dark:text-white">
          编辑用户
        </h2>
        <div class="space-y-4">
          <div>
            <label class="mb-1 block text-sm text-gray-700 font-medium dark:text-gray-300">
              用户名
            </label>
            <BaseInput
              v-model="editForm.username"
              placeholder="请输入用户名"
            />
          </div>
          <div>
            <label class="mb-1 block text-sm text-gray-700 font-medium dark:text-gray-300">
              密码
            </label>
            <BaseInput
              v-model="editForm.password"
              placeholder="请输入密码"
            />
          </div>
          <div>
            <label class="mb-1 block text-sm text-gray-700 font-medium dark:text-gray-300">
              账号额度
            </label>
            <BaseInput
              v-model.number="editForm.accountLimit"
              type="number"
              min="1"
              placeholder="可添加的账号数量"
            />
            <p class="mt-1 text-xs text-gray-500 dark:text-gray-400">
              用户最多可添加的农场账号数量
            </p>
          </div>
          <div>
            <label class="mb-1 block text-sm text-gray-700 font-medium dark:text-gray-300">
              过期时间
            </label>
            <div class="flex items-center gap-3">
              <input
                v-model="editForm.isPermanent"
                type="checkbox"
                class="border-gray-300 rounded text-blue-600 focus:ring-blue-500"
              >
              <span class="text-sm text-gray-600 dark:text-gray-400">永久有效</span>
            </div>
            <input
              v-if="!editForm.isPermanent"
              v-model="editForm.expiresAt"
              type="datetime-local"
              class="mt-2 w-full border border-gray-200 rounded-lg bg-white px-3 py-2 text-sm dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 dark:text-white focus:outline-none focus:ring-1 focus:ring-blue-500"
            >
          </div>
        </div>
        <div class="mt-6 flex justify-end space-x-3">
          <BaseButton variant="secondary" @click="showEditModal = false">
            取消
          </BaseButton>
          <BaseButton
            variant="primary"
            :disabled="editLoading"
            @click="handleEdit"
          >
            {{ editLoading ? '保存中...' : '保存' }}
          </BaseButton>
        </div>
      </div>
    </div>
  </div>
</template>
