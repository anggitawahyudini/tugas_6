<template>
  <div id="app">
    <div class="header">
      <h1>Pengelolaan Artikel <span class="icon">📚📝</span></h1>
    </div>
    <div class="container">
      <div class="form-section">
        <input type="text" v-model="newArticle.title" placeholder="Judul artikel">
        <textarea v-model="newArticle.content" placeholder="Isi artikel"></textarea>
        <input type="file" @change="onImageChange">
        <button @click="saveData">Simpan💾</button>
      </div>
      <button class="load-button" @click="loadData">Load Data🔄</button>
      <div v-if="dataLoaded" class="article-list">
        <div v-if="loading" class="loader"></div>
        <div v-else>
          <table class="styled-table">
            <thead>
              <tr>
                <th>Judul</th>
                <th>Isi</th>
                <th>Gambar</th>
                <th>Tanggal</th>
                <th>Aksi</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="article in articles" :key="article.id">
                <td>{{ article.title }}</td>
                <td>{{ article.content }}</td>
                <td>
                  <img v-if="article.image" :src="article.image" class="article-image" />
                </td>
                <td>{{ formatDate(article.date) }}</td>
                <td class="article-buttons">
                  <button @click="editArticle(article)">✏️ Edit</button>
                  <button @click="deleteArticle(article.id)">🗑️ Hapus</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div v-if="editMode" class="edit-section">
        <input type="text" v-model="editedArticle.title" placeholder="Judul artikel">
        <textarea v-model="editedArticle.content" placeholder="Isi artikel"></textarea>
        <input type="file" @change="onEditImageChange">
        <button @click="updateArticle">Update</button>
        <button @click="cancelEdit">Batal</button>
      </div>
    </div>
    <div class="footer">
      <p>&copy; 2024 Pengelolaan Artikel.</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      articles: [],
      dataLoaded: false,
      loading: false,
      newArticle: {
        title: '',
        content: '',
        image: '',
        date: ''
      },
      editedArticle: {
        id: '',
        title: '',
        content: '',
        image: '',
        date: ''
      },
      editMode: false
    };
  },
  methods: {
    loadData() {
      this.loading = true;
      axios.get('http://localhost:3000/articles')
        .then(response => {
          this.articles = response.data;
          this.dataLoaded = true;
          this.loading = false;
        })
        .catch(error => {
          console.error("There was an error loading the data!", error);
          this.loading = false;
        });
    },
    saveData() {
      const newArticleWithDate = { ...this.newArticle, date: new Date().toISOString() };
      axios.post('http://localhost:3000/articles', newArticleWithDate)
        .then(response => {
          console.log("Data saved successfully!", response.data);
          this.newArticle.title = '';
          this.newArticle.content = '';
          this.newArticle.image = '';
          this.loadData(); // Reload data after saving
        })
        .catch(error => {
          console.error("There was an error saving the data!", error);
        });
    },
    deleteArticle(articleId) {
      axios.delete(`http://localhost:3000/articles/${articleId}`)
        .then(response => {
          console.log("Article deleted successfully!", response.data);
          this.loadData(); // Reload data after deleting
        })
        .catch(error => {
          console.error("There was an error deleting the article!", error);
        });
    },
    editArticle(article) {
      this.editMode = true;
      this.editedArticle.id = article.id;
      this.editedArticle.title = article.title;
      this.editedArticle.content = article.content;
      this.editedArticle.image = article.image;
      this.editedArticle.date = article.date;
    },
    updateArticle() {
      axios.put(`http://localhost:3000/articles/${this.editedArticle.id}`, this.editedArticle)
        .then(response => {
          console.log("Article updated successfully!", response.data);
          this.editMode = false;
          this.editedArticle.id = '';
          this.editedArticle.title = '';
          this.editedArticle.content = '';
          this.editedArticle.image = '';
          this.editedArticle.date = '';
          this.loadData(); // Reload data after updating
        })
        .catch(error => {
          console.error("There was an error updating the article!", error);
        });
    },
    cancelEdit() {
      this.editMode = false;
      this.editedArticle.id = '';
      this.editedArticle.title = '';
      this.editedArticle.content = '';
      this.editedArticle.image = '';
      this.editedArticle.date = '';
    },
    onImageChange(event) {
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        this.newArticle.image = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    onEditImageChange(event) {
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        this.editedArticle.image = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    formatDate(date) {
      if (date) {
        return new Date(date).toLocaleDateString();
      }
      return '';
    }
  }
};
</script>

<style>
body {
  font-family: 'Roboto', sans-serif;
  background-color: #f4f4f4;
  margin: 0;
  padding: 0;
}

.header {
  background-color: #ffcccb;
  padding: 20px;
  text-align: center;
  border-bottom: 2px solid #ff99cc;
}

.header h1 {
  font-size: 2.5em;
  margin: 0;
  color: #fff;
}

.header .icon {
  font-size: 1.2em;
  margin-left: 10px;
}

.container {
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h2 {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin: 0;
}

p {
  font-size: 16px;
  color: #666;
  margin: 0;
}

input[type="text"],
textarea {
  width: calc(100% - 22px);
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #ffebee;
}

button {
  padding: 10px 20px;
  background-color: #ff99cc;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #ff6699;
}

/* Align buttons */
.article-buttons {
  display: flex;
  gap: 10px;
}

/* Load button */
.load-button {
  background-color: #ffcccb;
  margin-top: 20px;
}

.load-button:hover {
  background-color: #ff9999;
}

/* Edit and Delete buttons */
.article-buttons button {
  margin-left: 0;
}

.article-buttons button:hover {
  background-color: #ff9999;
}

/* Loading animation */
.loader {
  border: 6px solid #f3f3f3;
  /* Light grey */
  border-top: 6px solid #ff99cc;
  /* Pink */
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin: 20px auto;
  /* Centered loader */
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

@media screen and (max-width: 768px) {
  .container {
    padding: 10px;
  }

  h2 {
    font-size: 20px;
  }

  p {
    font-size: 14px;
  }

  input[type="text"],
  textarea,
  button {
    width: calc(100% - 22px);
  }
}

/* Styling for the table */
table.styled-table {
  width: 100%;
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 18px;
  text-align: left;
}

table.styled-table thead tr {
  background-color: #ff99cc;
  color: #ffffff;
  text-align: left;
  font-weight: bold;
}

table.styled-table th,
table.styled-table td {
  padding: 12px 15px;
}

table.styled-table tbody tr {
  border-bottom: 1px solid #dddddd;
}

table.styled-table tbody tr:nth-of-type(even) {
  background-color: #f3f3f3;
}

table.styled-table tbody tr:last-of-type {
  border-bottom: 2px solid #ff99cc;
}

table.styled-table tbody tr.active-row {
  font-weight: bold;
  color: #009879;
}

.edit-section {
  margin-top: 20px;
}

.form-section {
  margin-bottom: 20px;
}

/* Styling for images */
.article-image {
  max-width: 100px;
  border-radius: 5px;
}

.footer {
  background-color: #ffcccb;
  padding: 20px;
  text-align: center;
  border-top: 2px solid #ff99cc;
  margin-top: 20px;
}

.footer p {
  margin: 0;
  color: #fff;
}

/* Style for file input container */
.form-section input[type="file"] {
  display: block;
  margin-top: 10px;
}

/* Style for submit button */
.form-section button[type="submit"] {
  display: inline-block;
  vertical-align: top;
  margin-top: 10px;
}

/* Style for cancel button */
.form-section button[type="button"] {
  display: inline-block;
  vertical-align: top;
  margin-top: 10px;
  background-color: #ccc;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 10px 20px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.form-section button[type="button"]:hover {
  background-color: #999;
}
</style>