<template>
  <n-config-provider :theme="naiveTheme">
    <n-message-provider>
      <n-loading-bar-provider>
        <n-notification-provider>
          <n-dialog-provider>
            <div id="app" v-if="unlocked">
              <router-view />
            </div>

            <div v-else class="lock-screen">
              <div class="light-effect"></div>
              <div class="lock-card">
                <div class="lock-icon">
                  <svg viewBox="0 0 24 24" width="32" height="32" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round">
                    <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
                    <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
                  </svg>
                </div>
                <h2>安全验证</h2>
                <p class="subtitle">请输入访问密码</p>

                <!-- 文本输入框，可直接看到输入内容 -->
                <n-input
                  v-model:value="inputPwd"
                  type="text"
                  placeholder="输入密码"
                  @keyup.enter="doUnlock"
                  class="pwd-input"
                />

                <n-button type="primary" block @click="doUnlock" class="submit-btn">
                  验证并进入
                </n-button>
              </div>
            </div>
          </n-dialog-provider>
        </n-notification-provider>
      </n-loading-bar-provider>
    </n-message-provider>
  </n-config-provider>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from "vue";
import { darkTheme } from "naive-ui";
import { useTheme } from "@/composables/useTheme";

const { isDark, initTheme, setupSystemThemeListener, updateReactiveState } = useTheme();
const naiveTheme = computed(() => isDark.value ? darkTheme : null);

// 初始化时读取本地存储的验证状态（永不过期）
const unlocked = ref(localStorage.getItem('app_unlocked') === 'true');
const inputPwd = ref("");

// 哈希计算函数（保持原有逻辑）
function h(s) {
  let k = 0;
  for (let i = 0; i < s.length; i++) {
    k = Math.imul(k ^ s.charCodeAt(i), 7);
    k ^= k >>> 16;
  }
  return ("0000000" + (k >>> 0).toString(16)).slice(-8);
}

// 核心修改：校验值替换为“夜神月”对应的哈希
function x(t) {
  // 夜神月 + a/b/c 对应的哈希值
  const r1 = "02ddb0d0";  // h("夜神月"+"a") 计算结果
  const r2 = "02ddb325";  // h("夜神月"+"b") 计算结果
  const r3 = "02ddb322";  // h("夜神月"+"c") 计算结果
  const t1 = h(t + "a");
  const t2 = h(t + "b");
  const t3 = h(t + "c");
  return t1 === r1 && t2 === r2 && t3 === r3;
}

// 验证密码并存储永不过期的状态
function doUnlock() {
  const v = inputPwd.value.trim();
  if (x(v)) {
    unlocked.value = true;
    // 存储验证状态，永不过期
    localStorage.setItem('app_unlocked', 'true');
  } else {
    alert("密码错误");
  }
}

// 手动退出验证（可选）
function logout() {
  unlocked.value = false;
  localStorage.removeItem('app_unlocked');
}

// 主题相关逻辑
const handleThemeChange = () => {
  updateReactiveState();
  setTimeout(() => updateReactiveState(), 50);
};

onMounted(() => {
  initTheme();
  setupSystemThemeListener();
  window.addEventListener("theme-change", handleThemeChange);
  updateReactiveState();
});

onUnmounted(() => {
  window.removeEventListener("theme-change", handleThemeChange);
});
</script>

<style>
:root {
  --app-bg: linear-gradient(135deg, #3a1c71 0%, #d76d77 50%, #ffaf7b 100%);
  --text: #1d1d1f;
  --text-sub: #86868b;
  --card: rgba(255, 255, 255, 0.9);
  --border: rgba(0, 0, 0, 0.08);
  --glow: rgba(93, 95, 239, 0.3);
}

.dark {
  --app-bg: linear-gradient(135deg, #0f0f23 0%, #1a1a2e 50%, #252941 100%);
  --text: #f9fafb;
  --text-sub: #9ca3af;
  --card: rgba(22, 22, 33, 0.85);
  --border: rgba(255, 255, 255, 0.08);
  --glow: rgba(130, 100, 255, 0.4);
}

html.dark, html[data-theme="dark"] { color-scheme: dark; }

#app {
  min-height: 100vh;
  background: var(--app-bg);
  transition: background 0.3s ease;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  height: 100%;
  font-family: "SF Pro Display", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}

.lock-screen {
  width: 100vw;
  height: 100vh;
  background: var(--app-bg);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  position: relative;
  overflow: hidden;
}

.light-effect {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle at center, var(--glow) 0%, transparent 50%);
  animation: rotate 20s linear infinite;
  opacity: 0.3;
}

@keyframes rotate {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.lock-card {
  width: 100%;
  max-width: 380px;
  padding: 44px 36px;
  background: var(--card);
  border-radius: 24px;
  border: 1px solid var(--border);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  box-shadow: 0 25px 50px rgba(0,0,0,0.15), 0 0 0 1px var(--border);
  text-align: center;
  animation: pop 0.5s ease forwards;
  position: relative;
  z-index: 2;
}

@keyframes pop {
  0% { opacity: 0; transform: translateY(30px) scale(0.94); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
}

.lock-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 68px;
  height: 68px;
  border-radius: 50%;
  background: rgba(93, 95, 239, 0.12);
  color: #5d5fef;
  margin-bottom: 22px;
  box-shadow: 0 0 20px var(--glow);
}

.lock-card h2 {
  font-size: 24px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 6px;
}

.lock-card .subtitle {
  font-size: 15px;
  color: var(--text-sub);
  margin-bottom: 32px;
}

.pwd-input {
  margin-bottom: 20px;
}

.submit-btn {
  height: 48px;
  font-weight: 600;
  border-radius: 12px;
  transition: all 0.2s ease;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(93,95,239,0.3);
}
</style>