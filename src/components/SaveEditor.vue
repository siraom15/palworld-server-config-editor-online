<script setup>
import { reactive, ref } from "vue";

const handleUpload = async (e) => {
  const target = e.target;
  const file = target.files?.[0];
  if (!file) return;
  await getSettingFromFile(file);
};

const settings = ref(null);

const getSettingFromFile = async (file) => {
  const settingString = await readFile(file);
  if (!settingString) return;

  const regex = /OptionSettings=\((.*?)\)/;
  const match = regex.exec(settingString);

  if (match) {
    let tempObj = {};
    match[1].split(",").forEach((setting) => {
      const [key, value] = setting.split("=");
      tempObj[key] = value;
    });
    settings.value = tempObj;
    error.show = false;
    error.text = "";
    success.show = true;
    success.text = "Settings loaded successfully.";
  } else {
    error.show = true;
    error.text = "File does not contain settings.";
  }
};

const readFile = (file) => {
  return new Promise((resolve, reject) => {
    const fileReader = new FileReader();
    fileReader.onload = () => {
      const fileContent = fileReader.result;
      resolve(fileContent);
    };
    fileReader.onerror = () => {
      reject(null);
    };
    fileReader.readAsText(file);
  });
};

const error = reactive({
  show: false,
  text: "",
});
const success = reactive({
  show: false,
  text: "",
});

const saveFile = () => {
  const settingsString = Object.entries(settings.value)
    .map(([key, value]) => `${key}=${value}`)
    .join(",");
  const newSettings = `[/Script/Pal.PalGameWorldSettings]\nOptionSettings=(${settingsString})`;

  const blob = new Blob([newSettings], { type: "text/plain" });
  const url = URL.createObjectURL(blob);

  const a = document.createElement("a");
  a.href = url;
  a.download = "PalWorldSettings.ini";
  a.click();
  URL.revokeObjectURL(url);
};
</script>

<template>
  <!-- Step 1 -->
  <div class="p-20 pb-0 pt-0 mt-10" @drop.prevent="handleUpload">
    <div
      class="p-4 mb-4 text-sm text-red-800 rounded-lg bg-red-50 dark:bg-gray-800 dark:text-red-400"
      role="alert"
      v-if="error.show"
    >
      <span class="font-medium">Error</span> {{ error.text }}
    </div>
    <div
      class="p-4 mb-4 text-sm text-green-800 rounded-lg bg-green-50 dark:bg-gray-800 dark:text-green-400"
      role="alert"
      v-if="success.show"
    >
      <span class="font-medium">Success</span> {{ success.text }}
    </div>
    <p class="font-bold">Step 1: Select Save File</p>

    <div class="flex items-center justify-center w-full">
      <label
        for="dropzone-file"
        class="flex flex-col items-center justify-center w-full h-64 border-2 border-gray-300 border-dashed rounded-lg cursor-pointer bg-gray-50 dark:hover:bg-bray-800 dark:bg-gray-700 hover:bg-gray-100 dark:border-gray-600 dark:hover:border-gray-500 dark:hover:bg-gray-600"
      >
        <div class="flex flex-col items-center justify-center pt-5 pb-6">
          <svg
            class="w-8 h-8 mb-4 text-gray-500 dark:text-gray-400"
            aria-hidden="true"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 20 16"
          >
            <path
              stroke="currentColor"
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M13 13h3a3 3 0 0 0 0-6h-.025A5.56 5.56 0 0 0 16 6.5 5.5 5.5 0 0 0 5.207 5.021C5.137 5.017 5.071 5 5 5a4 4 0 0 0 0 8h2.167M10 15V6m0 0L8 8m2-2 2 2"
            />
          </svg>
          <p class="mb-2 text-sm text-gray-500 dark:text-gray-400">
            <span class="font-semibold">Click to upload</span> or drag and drop
          </p>
          <p class="text-xs text-gray-500 dark:text-gray-400">
            SVG, PNG, JPG or GIF (MAX. 800x400px)
          </p>
        </div>
        <input
          id="dropzone-file"
          type="file"
          class="hidden"
          @change="handleUpload"
        />
        <!-- <input type="file" name="save-file" id="save-file" @change="handleUpload" /> -->
      </label>
    </div>
    <p>
      Save file location
      <code
        >steamapps\common\PalServer\Pal\Saved\Config\WindowsServer\PalWorldSettings.ini
      </code>
    </p>
  </div>

  <!-- Step 2 -->
  <div class="p-20 pb-0 pt-0 mt-10" v-if="settings">
    <p class="font-bold">Step 2: Edit save file</p>
    <p>
      More information about settings :
      <a
        class="text-blue-700 hover:underline"
        target="_blank"
        href="https://tech.palworldgame.com/settings-and-operation/configuration"
        >Visit official website</a
      >
    </p>

    <div v-for="(value, key) in settings" :key="key">
      <div class="max-w-sm mt-5">
        <label
          :for="key"
          class="block mb-2 text-sm font-medium text-gray-900 dark:text-white"
          >{{ key }}</label
        >
        <div class="flex">
          <span
            class="inline-flex items-center px-3 text-sm text-gray-900 bg-gray-200 border border-e-0 border-gray-300 rounded-s-md dark:bg-gray-600 dark:text-gray-400 dark:border-gray-600"
          >
            {{ key }}
          </span>
          <input
            type="text"
            :id="key"
            :value="value"
            class="rounded-none rounded-e-lg bg-gray-50 border border-gray-300 text-gray-900 focus:ring-blue-500 focus:border-blue-500 block flex-1 min-w-0 w-full text-sm p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            placeholder="Bonnie Green"
          />
        </div>
      </div>
    </div>
  </div>

  <!-- Step3 -->
  <div class="p-20 pb-0 pt-0 mt-10" v-if="settings">
    <p class="font-bold">Step 3: Save File</p>
    <p class="text-xs">
      Click the button below to save the file. You can overwrite the existing file.
    </p>
    <button
      type="button"
      @click="saveFile"
      class="text-white mt-5 bg-blue-700 hover:bg-blue-800 focus:outline-none focus:ring-4 focus:ring-blue-300 font-medium rounded-full text-sm px-5 py-2.5 text-center me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
    >
      Save
    </button>
  </div>
</template>
