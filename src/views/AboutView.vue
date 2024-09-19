<script lang="ts">
import { ref } from 'vue';
import PouchDB from 'pouchdb'

declare interface Post {
  id: string,
  doc: {
    post_name: string,
    post_content: string,
    attributes: {
      creation_date: string
    }
  }
}

export default {
  data() {
    return {
      total: 0,
      postsData: [] as Post[],
      storage: null as PouchDB.Database | null,
    };
  },

  mounted() {
    this.initDatabase();
    this.fetchData();
  },

  methods: {

    fetchData() {
      const storage = ref(this.storage);
      const self = this;
      if (storage.value) {
        (storage.value).allDocs({
          include_docs: true,
          attachments: true
        }).then(function (result: any) {
          console.log('fetchData success', result.rows);
          self.postsData = result.rows;
        }.bind(this)).catch(function (error: any) {
          console.log('fetchData error', error);
        });
      }
    },

    initDatabase() {
      const $storage = new PouchDB('http://admin:admin@localhost:5984/post');
      if ($storage) {
        console.log("Connected to collection 'post'");
      } else {
        console.warn("Something went wrong");
      }
      this.storage = $storage;
    }
  },

}
</script>

<template>
  <h1>Nombre de post: {{ postsData.length }}</h1>
  <ul>
    <li v-for="post in postsData" :key="post.id">
      <div class="ucfirst">{{ post.doc.post_name }}<em style="font-size: x-small;"
          v-if="post.doc.attributes?.creation_date">
          - {{ post.doc.attributes?.creation_date }}
        </em>
      </div>

    </li>
  </ul>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}

.ucfirst:first-letter {
  text-transform: uppercase;

}
</style>
