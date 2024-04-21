<script>
import { initializeApp } from "firebase/app";
import { getFirestore, doc, getDoc, setDoc } from "firebase/firestore";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Copy } from "lucide-vue-next";

export default {
  name: "Shortn",
  data() {
    return {
      firebaseConfig: {
        apiKey: "AIzaSyBhJh8sMYpT-4c1CdtZbAqKUUKVsx9kIrI",
        authDomain: "shortn-e3617.firebaseapp.com",
        projectId: "shortn-e3617",
        storageBucket: "shortn-e3617.appspot.com",
        messagingSenderId: "27099966214",
        appId: "1:27099966214:web:8bbc069667f787ee064c87",
      },
      urlIndex: [],
      keyIndex: [],
      toShorten: "",
      shortenedURL: "",
    };
  },
  components: {
    Input,
    Button,
    Copy,
  },
  async mounted() {
    //initialize firebase
    let app = initializeApp(this.firebaseConfig);
    let db = getFirestore(app);
    //get index
    let docRef = doc(db, "data", "data");
    let docSnap = await getDoc(docRef);
    if (docSnap.exists()) {
      this.urlIndex = docSnap.data().urlIndex;
      this.keyIndex = docSnap.data().keyIndex;
    } else {
      alert(
        "An error occurred while fetching the index. Please try again later."
      );
    }
    //
    let path = window.location.href.split("/");
    path = path[3];
    //check if this is a shortened url
    if (path.length > 0) {
      //redirect to the original url
      docRef = doc(db, "data", path);
      docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        window.open("https://" + docSnap.data().url, "_self");
      }
    }
  },
  methods: {
    generateKey() {
      //get variables
      let charset =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+*#%&=?!$£-_.:,;";
      let charsetLength = charset.length;
      let charCount = 5;
      //generate a random key
      let key = "";
      let found = false;
      while (!found) {
        for (let i = 0; i < charCount; i++) {
          key += charset.charAt(Math.floor(Math.random() * charsetLength));
        }
        if (!this.keyIndex.includes(key)) {
          found = true;
        }
      }
      return key;
    },
    async shortenURL(url) {
      //validate if real url
      if (
        !url.match(
          /(http(s)?:\/\/.)?(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)/g
        )
      ) {
        alert("Please enter a valid URL.");
      } else {
        //clean from unwanted details
        url = url.replace("https://", "");
        url = url.replace("http://", "");
        url = url.replace("www.", "");
        //check if url has already been shorted
        if (this.urlIndex.includes(url)) {
          this.shortenedURL =
            window.location + this.keyIndex[this.urlIndex.indexOf(url)];
        } else {
          //else shorten
          let key = this.generateKey();
          this.shortenedURL = window.location + key;
          //initialize firebase
          let app = initializeApp(this.firebaseConfig);
          let db = getFirestore(app);
          //set new doc
          this.urlIndex.push(url);
          this.keyIndex.push(key);
          await setDoc(doc(db, "data", "data"), {
            urlIndex: this.urlIndex,
            keyIndex: this.keyIndex,
          });
          await setDoc(doc(db, "data", key), {
            url: url,
          });
        }
      }
    },
    copyToClipboard(toCopy) {
      navigator.clipboard.writeText(toCopy);
    },
    reset() {
      this.toShorten = "";
      this.shortenedURL = "";
    },
  },
};
</script>

<template>
  <div v-if="shortenedURL.length <= 0" class="grid h-screen place-items-center">
    <div class="grid w-1/3 place-items-center">
      <p>Enter a URL to shorten:</p>
      <Input
        type="url"
        placeholder="eg. 'apple.com'"
        v-model="toShorten"
        class="m-1 text-center"
      />
      <Button class="m-1" @click="shortenURL(toShorten)">Shorten</Button>
    </div>
  </div>
  <div v-else class="grid h-screen place-items-center">
    <div class="grid w-1/3 place-items-center">
      <p class="m-1">Shortened URL:</p>
      <div class="flex w-full">
        <Input
          v-model="shortenedURL"
          class="m-1 text-center"
          style="cursor: text"
          disabled
        />
        <Button
          variant="outline"
          class="m-1"
          @click="copyToClipboard(shortenedURL)"
          ><Copy></Copy
        ></Button>
      </div>
      <Button class="m-1" @click="reset()">Shorten another</Button>
    </div>
  </div>
</template>
