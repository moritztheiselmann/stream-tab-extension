<script setup lang="ts">
  import { ref, computed, onMounted, onUnmounted } from 'vue';
  
  import { Messages } from './messages';
  import { sendMessageToBackground, sendMessageToContentScript } from './messenger';
  
  const startScreenCapture = async() => {
    try {
      const response = await sendMessageToBackground(Messages.SS_UI_REQUEST, { message: 'Initialise Screen Capture' });
      console.log(`response: ${response}`);
    }
    catch (err) {
      console.error(`error: ${err}`);
    }
  }
  
  const startTabCapture = async() => {
    try {
      const tab = await getCurrentTab();
      const tabID = tab.id;
      if (!tabID) {
        return;
      }
      chrome.tabCapture.getMediaStreamId({consumerTabId: tabID}, (streamID) => {
        const response = sendMessageToContentScript(tabID, Messages.SS_TAB_REQUEST, 
          { 
            message: 'Initialise Tab Capture',
            streamID: streamID
          }
        );
        console.log(`response: ${response}`);
      });

    }
    catch (err) {
      console.error(`error: ${err}`);
    }
  }

  const stopCapturing = async() => {
    try {
      const response = await sendMessageToBackground(Messages.SS_UI_CANCEL, { message: 'Stop Screen Capture' });
      console.log(`response: ${response}`);
      setActive(false);
    }
    catch (err) {
      console.error(`error: ${err}`);
    }
  }
  
  // const icons = ref({
  //   active: '',
  //   inactive: ''
  // });
  const active = ref(<boolean>false);
  const setActive = (value : boolean ) => {
    chrome.storage.local.set({ 'toggleCapturing': value })
    .then(() => {
      console.log(`toggleCapturing is set to + ${value}`);
    });

    // chrome.browserAction.setIcon({
    //   path: icons.value[active.value ? 'active' : 'inactive']
    // });
  }
  
  onMounted(() => {
      chrome.storage.local.get(['toggleCapturing'], (result) => {
        console.log(`got data from local storage: ${result.toggleCapturing}`);
        active.value = result.toggleCapturing;
      })

      startTabCapture();
  });

  chrome.storage.onChanged.addListener((changes, namespace) => {
    if (namespace === 'local') {
      if (changes.toggleCapturing) {
        active.value = changes.toggleCapturing.newValue;
        console.log(active.value);
      }
    }
  });

  const getCurrentTab = async() => {
    const querryOptions = {
      active: true,
      lastFocusedWindow: true,
      currentWindow: true
    };

    const [tab] = await chrome.tabs.query(querryOptions); 
    
    return tab;
  }
</script>

<template>
  <div>
    <button @click="startScreenCapture">
      Record Screen
    </button>
    
    <button @click="startTabCapture">
      Capture Tab
    </button>
    <!-- <button 
      v-if="!active" 
      @click="startCapturing">
      Start Capturing
    </button>
    <button 
      v-else
      @click="stopCapturing">
      Stop Capturing
    </button> -->
    <p>
      Capturing: {{ active }}
    </p>
  </div>
</template>