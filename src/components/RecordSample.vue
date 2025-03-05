<template>
  <div class="flex flex-center">
    <button
      class="margin-small"
      v-if="!recordIsAllowed"
      @click="requestRecording"
    >
      <span>Request mic access</span>
    </button>
    <button
      class="padding-extra-small q-mr-sm"
      :disabled="disable"
      v-if="recordIsAllowed && currentStatus === 'NONE'"
      @click="startRecording"
    >
      <span>Start Recording</span>
    </button>
    <button
      class="padding-extra-small q-mr-sm"
      v-if="recordIsAllowed && currentStatus === 'RECORDING'"
      @click="stopRecording"
    >
      <span>Stop Recording</span>
    </button>
    <button
      v-if="currentStatus === 'RECORDING'"
      class="margin-x-small bg-grey-4 no-stroke"
      @click="pauseRecording"
    >
      <span>Pause</span>
    </button>
    <button
      class="padding-extra-small"
      v-if="recordIsAllowed && currentStatus === 'PAUSED'"
      @click="resumeRecording"
    >
      <span>Resume Recording</span>
    </button>

    <div v-if="recordingData">
      <audio :src="recordingData.audioURL" controls></audio>
      <div>Name: {{ recordingData.name }}</div>
      <div>Duration: {{ recordingData.duration }}ms</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { VoiceRecorder } from 'capacitor-voice-recorder';

const recordIsAllowed = ref(false);
const currentStatus = ref('NONE');
const recordingName = ref('Untitled');
const organizationId = ref(null);
const recordingData = ref(null);
const disable = ref(false);

onMounted(() => {
  requestRecording();
  syncStatus();
});

async function requestRecording() {
  if (recordIsAllowed.value) {
    return true;
  }

  const canRecord = await VoiceRecorder.canDeviceVoiceRecord();
  if (!canRecord.value) {
    console.error('Device does not support audio recording. Please check your app permissions');
    return false;
  } else {
    const result = await VoiceRecorder.requestAudioRecordingPermission();
    if (result.value) {
      recordIsAllowed.value = true;
      return true;
    } else {
      console.error('Cannot access microphone. Please check your app settings');
      recordIsAllowed.value = false;
      return false;
    }
  }
}

async function startRecording(name = 'Untitled', orgId = null) {
  recordingName.value = name;
  organizationId.value = orgId;
  
  try {
    await VoiceRecorder.startRecording({
      smallIcon: 'ic_notification',
      useForegroundService: true,
    });
    currentStatus.value = 'RECORDING';
    console.log('Recording started');
  } catch (e) {
    console.error('Recording start failed', e);
    await syncStatus();
  }
}

async function pauseRecording() {
  try {
    await VoiceRecorder.pauseRecording();
    currentStatus.value = 'PAUSED';
  } catch (e) {
    console.error('Pause recording failed', e);
    await syncStatus();
  }
}

async function resumeRecording() {
  try {
    await VoiceRecorder.resumeRecording();
    currentStatus.value = 'RECORDING';
  } catch (e) {
    console.error('Resume recording failed', e);
    await syncStatus();
  }
}

async function stopRecording() {
  try {
    const result = await VoiceRecorder.stopRecording();
    currentStatus.value = 'NONE';
    
    if (!!result.value?.recordDataBase64) {
      const recording = {
        status: 'local',
        id: Date.now().toString(),
        createdAt: new Date(),
        name: recordingName.value,
        organizationId: organizationId.value,
        audioURL: `data:${result.value.mimeType};base64,${result.value.recordDataBase64}`,
        mimeType: result.value.mimeType,
        duration: result.value.msDuration,
      };
      recordingData.value = recording;
      return recording;
    }
  } catch (e) {
    console.error('Stop recording failed', e);
    await syncStatus();
  }
  recordingName.value = 'Untitled';
  organizationId.value = null;
}

function resetStatus() {
  currentStatus.value = 'NONE';
}

async function syncStatus() {
  try {
    const { status } = await VoiceRecorder.getCurrentStatus();
    if (status === 'NONE') {
      resetStatus();
    } else {
      currentStatus.value = status;
    }
  } catch (e) {
    console.error('Error getting recording status', e);
    resetStatus();
  }
}

defineProps({
  initialName: {
    type: String,
    default: 'Untitled',
  },
  initialOrganizationId: {
    type: String,
    default: null,
  }
});

defineEmits(['recording-complete']);
</script>
