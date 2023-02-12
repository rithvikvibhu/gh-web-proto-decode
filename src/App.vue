<template>
  <div id="app" class="flex-grow">
    <!-- File Select -->
    <section
      class="flex justify-around mx-20 mt-8 p-6 bg-blue-100 rounded-md border-2 border-blue-400"
    >
      <h2 class="text-2xl">Select proto file</h2>
      <form @submit.prevent="readFile">
        <input type="file" name="protofile" id="protofile" />
        <input
          type="submit"
          value="Decode"
          class="px-2 py-1 bg-blue-300 hover:bg-blue-400 rounded border border-blue-400 bg-white"
        />
      </form>
    </section>

    <!-- Table -->
    <section
      v-if="showDevices"
      class="mx-20 mt-8 p-6 bg-blue-100 rounded-md border-2 border-blue-400"
    >
      <h2 class="text-2xl">Devices found</h2>
      <br />
      <table class="w-full table-auto">
        <thead>
          <tr>
            <td class="px-4 py-2 font-semibold">Sl No.</td>
            <td class="px-4 py-2 font-semibold">Device Type</td>
            <td class="px-4 py-2 font-semibold">Device Name</td>
            <td class="px-4 py-2 font-semibold">Token</td>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="device in devices"
            :key="device.id"
            :class="{ 'bg-blue-200': device.id % 2 }"
          >
            <td class="border px-4 py-2">{{ device.id }}</td>
            <td class="border px-4 py-2">{{ device.type || "-" }}</td>
            <td class="border px-4 py-2">{{ device.name || "-" }}</td>
            <td class="border px-4 py-2">{{ device.token || "-" }}</td>
          </tr>
          <tr v-if="!devices.length">
            <td class="px-4 py-4 text-center" colspan="4">
              No devices found :(
            </td>
          </tr>
        </tbody>
      </table>
    </section>

    <!-- Info section -->
    <section
      class="w-7/12 mx-auto mt-8 p-6 bg-white rounded border border-blue-400"
    >
      <h3 class="text-2xl">What is this?</h3>
      <p class="mt-2">
        Until recently, the Google Home app used to communicate with the device
        over port 8008 (HTTP) and did not require any authentication. Everything
        in the
        <a
          target="_blank"
          href="https://rithvikvibhu.github.io/GHLocalApi/"
          class="link"
          >unofficial documentation</a
        >
        worked as expected.
      </p>
      <p class="mt-2">
        A few months ago, Google pushed a new update to all GH devices and all
        endpoints started returning 403 (forbidden) errors. The app had switched
        over to port 8443 and HTTPS.
      </p>
      <p class="mt-2">
        One way to make it work again is to send a token with all requests. This
        token is stored in a file in the Google Home app's data directory, but
        in protobuf format which is hard to read.
      </p>
      <p class="mt-2">
        This website helps read that file and extract all the tokens.
      </p>
      <p class="mt-2">
        For more info, check out
        <a
          target="_blank"
          href="https://gist.github.com/rithvikvibhu/1a0f4937af957ef6a78453e3be482c1f"
          class="link"
          >this gist</a
        >
        and
        <a
          target="_blank"
          href="https://github.com/rithvikvibhu/GHLocalApi/issues/39"
          class="link"
          >this issue</a
        >.
      </p>

      <h3 class="text-2xl mt-6">
        Why should I upload my secret token file online?
      </h3>
      <p class="mt-2">
        Well, the script required NodeJs to run (which you can still do), but
        this site just makes it easier.<br />Also, everything happens in the
        browser, offline, and
        <span class="font-semibold"
          >no device info / token ever leaves your browser</span
        >.
      </p>
    </section>
  </div>
</template>

<script>
import { getData } from "rawproto";

export default {
  data() {
    return {
      showDevices: false,
      devices: []
    };
  },
  methods: {
    readFile(event) {
      this.showDevices = false;
      var file = event.target.firstElementChild.files[0];
      if (!file) alert("Pick a file to decode.");
      else {
        const reader = new FileReader();
        reader.onload = ev => {
          console.log(`[*] Parsing file...`);
          console.log(ev.target.result);
          var data = getData(buffer.Buffer.from(ev.target.result));
          console.log("[*] Extracting tokens...");
          var i = 1;
          data.forEach(val => {
            if (val["2"] && Array.isArray(val["2"])) {
              val["2"].forEach(val2 => {
                if (val2["7"] && Array.isArray(val2["7"])) {
                  var device = null;
                  var token = null;
                  try {
                    var deviceObject = getObjByKey(val2["7"], "17")["17"][0];
                    var deviceType = deviceObject
                      ? deviceObject["2"] || deviceObject["1"]
                      : null;
                    var deviceName = getObjByKey(val2["7"], "18")["18"][0]["2"];
                  } catch (err) {
                    console.log(err);
                  }
                  try {
                    var tokenObject = getObjByKey(val2["7"], "28");
                    token = tokenObject ? tokenObject["28"] : null;
                  } catch (err) {
                    console.log(err);
                  }
                  console.log("Device:\t", deviceName, "\nToken:\t", token);
                  this.devices.push({
                    id: i++,
                    type: deviceType,
                    name: deviceName,
                    token
                  });
                }
              });
            }
          });
          this.showDevices = true;
        };
        reader.onerror = err => console.log(err);
        reader.readAsArrayBuffer(file);
      }
    }
  }
};

function detailedLog(obj) {
  console.log(
    util.inspect(obj, {
      showHidden: false,
      depth: null
    })
  );
}

function getObjByKey(arr, key) {
  return arr.filter(v => v[key])[0];
}
</script>

<style>
</style>