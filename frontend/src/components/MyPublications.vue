<template>
  <div class="all">
    <!-- Publier un post -->

    <!-- Formulaire -->
    <div v-if="userId == Token.userId" class="publicationCard">
      <h2>Publiez...</h2>
      <form @submit.prevent="onSubmit" enctype="multipart/form-data">
        <div>
          <textarea
            v-model="text"
            name="text"
            id=""
            cols="30"
            rows="10"
            placeholder="Tapez votre texte ici"
          ></textarea>
        </div>

        <div class="send-it">
          <input id="file" type="file" ref="file" @change="onSelect" />

          <button @click="sendText" type="submit" class="btn btn-publier">
            Publier
          </button>
        </div>
        <p class="errorMessage" v-if="error">{{ error }}</p>
      </form>
    </div>

    <!-- Fin publier un post -->

    <!-- Afficher les publications du profil -->

    <div v-if="publications.length">
      <div
        v-for="publication in publications"
        :key="publication"
        class="publicationCard"
      >
        <!-- Si la publication est partagée -->

        <div
          v-if="
            publication.shared == 1 && publication.shared_userId == Token.userId
          "
          class="publicationDate shared"
        >
          <div class="userAndImage shared">
            <img
              :src="
                require(`./../../../backend/images/${publication.shared_userProfil_picture}`)
              "
              width="100"
              alt=""
            />
          </div>
          <p>partagé par {{ publication.shared_userName }}</p>
          <p class="datePub">Partagé le {{ publication.DATETIME_FR }}</p>
          <hr />
        </div>

        <!-- fin si publication partagée -->

        <div class="publicationDate">
          <div class="userAndImage">
            <img
              :src="
                require(`./../../../backend/images/${publication.profil_picture}`)
              "
              width="200"
              alt=""
            />
          </div>
          <p>Publié par {{ publication.user_send }}</p>
          <p class="datePub">Le {{ publication.DATETIME_FR }}</p>
        </div>

        <div v-if="publication.image" class="publicationPhoto">
          <img
            :src="require(`./../../../backend/images/${publication.image}`)"
            width="200"
            alt=""
          />
        </div>

        <p class="publicationText">{{ publication.post }}</p>

        <!-- Likes && dislikes -->
        <div class="likes">
          <div>
            <i
              @click="like_it(publication.postId)"
              class="fas fa-thumbs-up like"
            >
              <span class="like"> &#160; {{ publication.likes_number }} </span>
            </i>

            <i
              @click="deslike_it(publication.postId)"
              class="fas fa-thumbs-down deslike"
            >
              <span class="like">
                &#160; {{ publication.deslikes_number }}
              </span>
            </i>
          </div>

          <!-- Supprimer et modifier un post -->
          <div class="deleteModify">
            <!-- Supprimer un post -->

            <i
              v-if="publication.userId == Token.userId || Token.isAdmin == 1"
              @click="deletePost(publication.postId)"
              class="fas fa-trash delete"
            >
              <span> Supprimer </span></i
            >

            <!-- Modifier un post -->

            <i
              v-if="publication.userId == Token.userId"
              @click="btnModifyPost(publication.postId)"
              class="fas fa-pen-square modify"
              ><span> Modifier </span></i
            >
          </div>
        </div>

        <div
          v-if="okModifyPost && publication.postId == this.postId"
          class="publicationCard modifyPostCard"
        >
          <form @submit.prevent="onModify" enctype="multipart/form-data">
            <div>
              <textarea
                v-model="textModified"
                name="text"
                id=""
                cols="30"
                rows="10"
                placeholder=" Votre nouveau texte "
              >
              </textarea>
            </div>

            <div class="send-it">
              <div>
                <input type="file" ref="file" @change="onSelected" />
              </div>
              <div class="field">
                <button
                  @click="updatePost()"
                  type="submit"
                  class="btn btn-publier"
                >
                  Valider
                </button>
              </div>
            </div>
            <p v-if="error" class="errorMessage">{{ error }}></p>
          </form>
        </div>

        <!-- Commentaires -->

        <!-- 1/ Publier un commentaire -->
        <div class="commentCard">
          <div class="commentAndPartage likes share">
            <!-- Bouton Commenter -->
            <button class="cachedButton">
              <i
                class="far fa-comment-dots comment"
                @click="btnComment(publication.postId)"
              >
                <span> Commenter </span>
              </i>
            </button>

            <!-- Bouton Partager -->
            <button class="cachedButton">
              <i @click="share(publication)" class="fas fa-share-square share">
                <span>
                  Partager
                  <span class="shared_number">
                    {{ publication.shared_number }}</span
                  >
                </span>
              </i>
            </button>
          </div>

          <form
            v-if="okComment && publication.postId == this.postId"
            @submit.prevent=""
          >
            <textarea
              v-model="comment"
              cols="30"
              name="text"
              rows="10"
              placeholder=" Votre commentaire "
            >
            </textarea>
            <button
              v-if="comment"
              @click="createComment(publication.postId)"
              type="submit"
              class="btn btnComment"
            >
              Envoyer
            </button>
          </form>
        </div>
        <!--- Fin publier un commentaire    -->

        <!--  2/ Afficher tous les commentaires du postId-->
        <div class="allComments">
          <button
            v-if="publication.commentNumber"
            @click="getAllComments(publication.postId)"
          >
            Commentaires
            <span> {{ publication.commentNumber }} </span>
          </button>

          <div v-if="showComments && publication.postId == comments[0].postId">
            <div v-for="comment in comments" :key="comment">
              <div class="oneCommentCard">
                <div class="publicationDate">
                  <div class="userAndImage">
                    <img
                      :src="
                        require(`./../../../backend/images/${comment.profil_picture}`)
                      "
                      width="200"
                      alt=""
                    />
                  </div>
                  <p>{{ comment.user_send }}</p>
                  <p class="datePub">Publié le {{ comment.DATETIME_FR }}</p>
                </div>

                <!-- Masquer un commentaire -->
                <p v-if="comment.masked == 1" class="masked">
                  Ce commentaire est masqué par l'administrateur !
                </p>
                <div class="notShowComment">
                  <div
                    v-if="comment.masked == 0 || okMasked"
                    class="deleteComment"
                  >
                    <p>{{ comment.comment }}</p>

                    <!-- Supprimer un commentaire -->
                    <p>
                      <button class="cachedButton">
                        <i
                          title="Supprimer"
                          v-if="
                            comment.userId ==
                              $store.state.userConnected.userId ||
                            $store.state.userConnected.isAdmin == 1
                          "
                          @click="
                            deleteComment(comment.commentId, publication.postId)
                          "
                          class="fas fa-trash-alt"
                        ></i>
                      </button>
                    </p>
                    <!-- Fin supprimer un commentaire -->
                  </div>

                  <div>
                    <button class="cachedButton">
                      <i
                        title="Masquer"
                        @click="
                          maskComment(comment.commentId, publication.postId)
                        "
                        v-if="
                          $store.state.userConnected.isAdmin == 1 &&
                          comment.masked == 0
                        "
                        class="fas fa-eye-slash maskedIcone"
                      >
                      </i>
                    </button>

                    <button class="cachedButton">
                      <i
                        title="Voir"
                        @click="demaskComment(comment.commentId)"
                        v-if="
                          userConnected.isAdmin == 1 &&
                          comment.masked == 1 &&
                          !okMasked
                        "
                        class="fas fa-eye demaskedIcone"
                      ></i>
                    </button>
                  </div>
                </div>
                <hr />
              </div>
            </div>
          </div>
        </div>
        <button
          v-if="!afficherCommentaires && publication.commentNumber > 3"
          @click="getAllComments(publication.postId)"
        >
          Masquer les commentaires
        </button>

        <!--- Fin publier un commentaire    -->
      </div>
    </div>

    <div v-else>Il n'y a aucune publication à afficher !</div>
    <p class="authentified">{{ authentified }}</p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "MyPublications",

  props: {
    userConnected: {
      type: Object,
      default() {
        return {
          userConnected: "userConnected",
        };
      },
    },
  },

  data() {
    return {
      userId: "",
      text: "",
      file: "",
      publications: [],
      error: "",
      okModifyPost: false,
      postId: null,
      textModified: "",
      comment: "",
      okComment: false,
      okAllComments: true,
      commentNumber: 0,
      comments: [],
      showComments: false,
      status: 0,
      afficherCommentaires: true,
      authentified: "",
      Token: "",
      id: "",
      demasked: false,
      okMasked: false,
    };
  },

  created() {
    this.getAllMyPosts();
    this.userId = this.$route.params.id;
    this.messageThreeSeconds();
    this.error = "";
  },

  computed() {
    this.$route.params.id;
  },

  mounted() {
    this.getAllMyPosts();
  },

  methods: {
    messageThreeSeconds: function () {
      setTimeout(() => {
        this.error = "";
      }, 3000);
    },

    btnModifyPost(id) {
      this.okModifyPost = !this.okModifyPost;
      this.postId = id;
    },

    btnComment(id) {
      this.okComment = !this.okComment;
      this.postId = id;
    },

    //----  1- Afficher toutes mes publications ////////
    getAllMyPosts() {
      this.Token = JSON.parse(localStorage.getItem("Token"));

      if (this.Token.userId == this.$route.params.id) {
        this.id = this.Token.userId;
      } else {
        this.id = this.$route.params.id;
      }
      axios
        .get("http://localhost:3000/api/posts/" + this.id, {
          headers: {
            authorization: `bearer ${this.Token.token}`,
          },
        })

        .then((response) => {
          // Si requête non authentifiée
          if (response.data.disconnected) {
            this.authentified = response.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            this.publications = response.data;
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    ///// Fin afficher tous mes posts ////////

    // -2/----- Partager une publication ---//
    share(publication) {
      axios
        .post(
          "http://localhost:3000/api/posts/share",
          {
            publication1: publication,
            publication2: {
              shared: 1,
              shared_userId: this.userConnected.userId,
              shared_userName: this.userConnected.name,
              shared_userProfil_picture: this.userConnected.profil_picture,
            },
          },
          {
            headers: {
              authorization: `bearer ${this.Token.token}`,
            },
          }
        )
        .then((res) => {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            this.publications = res.data;
            this.getAllMyPosts();
          }
        })
        .catch((err) => {
          console.log(" Erreur:", err);
        });
      this.getAllMyPosts();
    },

    // ---- 3/- Publier un post //

    sendText(f) {
      if (!this.file && !this.text) {
        this.error = " Merci de joindre du texte ou une image à publier !";
        this.messageThreeSeconds();
        f.preventDefault();
      }
    },

    onSelect() {
      const allowedTypes = [
        "image/jpeg",
        "image/jpg",
        "image/png",
        "image/gif",
      ];
      const file = this.$refs.file.files[0];
      this.file = file;
      if (!allowedTypes.includes(file.type)) {
        this.error = "Merci de choisir un fichier PNG, JPG, JPEG ou GIF!";
        this.file = "";
      }
      if (file.size > 500000) {
        this.error = "Votre fichier est trop volumineux: 500kb max! ";
        this.file = "";
      }
    },

    async onSubmit() {
      const formData = new FormData();
      formData.append("file", this.file);
      formData.append("post", this.text);
      formData.append("user_send", this.userConnected.name);
      formData.append("userId", this.userConnected.userId);
      formData.append("profil_picture", this.userConnected.profil_picture);
      let self = this;

      axios
        .post("http://localhost:3000/api/posts", formData, {
          headers: {
            authorization: `bearer ${this.Token.token}`,
          },
        })

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.getAllMyPosts();
          }
        })
        .catch(function (err) {
          console.log("Erreur 1:", err);
        });
      this.text = "";
      self.getAllMyPosts();
      document.getElementById("file").value = "";
    },
    //---------- Fin publier un post ---------------//

    //---4/- Supprimer un post ------------ //
    deletePost(id) {
      const self = this;
      axios
        .delete("http://localhost:3000/api/posts/" + id, {
          headers: {
            authorization: `bearer ${this.Token.token}`,
          },
        })

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.getAllMyPosts();
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    // ----------- Fin supprimer un post ---------- //

    //-- 5/ ---------- Modifier un post -------------//
    onModify(f2) {
      if (!this.file && !this.textModified) {
        this.error = " Merci de joindre du texte ou une image pour envoyer !";
        this.messageThreeSeconds();
        f2.preventDefault();
        this.okModifyPost = !this.okModifyPost;
      }
    },
    onSelected() {
      const allowedTypes = [
        "image/jpeg",
        "image/jpg",
        "image/png",
        "image/gif",
      ];
      const file = this.$refs.file.files[0];
      this.file = file;
      if (!allowedTypes.includes(file.type)) {
        this.error = "Merci de choisir un fichier PNG, JPG, JPEG ou GIF!";
      }
      if (file.size > 500000) {
        this.error = "Votre fichier est trop volumineux: 500kb max! ";
      }
    },
    updatePost(f) {
      if (!this.textModified && !this.file.name) {
        f.preventDefault();
      }
      const formData = new FormData();
      if (this.textModified) {
        formData.append("textModified", this.textModified);
      }
      if (this.file) {
        formData.append("file", this.file);
      }
      const self = this;
      axios
        .put("http://localhost:3000/api/posts/" + this.postId, formData, {
          headers: {
            authorization: `bearer ${this.Token.token}`,
          },
        })

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.getAllMyPosts();
          }
        })
        .catch(function (err) {
          console.log("Erreur :", err);
        });

      this.okModifyPost = !this.okModifyPost;
      this.textModified = "";
      this.error = "";
      this.getAllMyPosts();
      location.reload();
    },
    // ---------- Fin modifier un post ------------//

    // --- 6/ --- Créer un commentaire -------------- //
    createComment(postId) {
      axios
        .post(
          "http://localhost:3000/api/comments",
          {
            comment: this.comment,
            postId: postId,
            userId: this.userConnected.userId,
            user_send: this.userConnected.name,
            profil_picture: this.userConnected.profil_picture,
          },
          {
            headers: {
              authorization: `bearer ${this.Token.token}`,
            },
          }
        )

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.okComment = false;
            self.getAllComments(postId);
            self.getAllMyPosts(postId);
          }
        })
        .catch(function (err) {
          console.log(err);
        });
      this.okComment = false;
      this.comment = "";
      this.getAllComments(postId);
      this.getAllMyPosts(postId);
    },
    // ------- Fin créer un commentaire --------- //

    // --7/ ----- Afficher tous les commentaires d'un post--------- //
    getAllComments(postId) {
      this.showComments = !this.showComments;
      const self = this;
      axios
        .get("http://localhost:3000/api/comments/" + postId, {
          headers: {
            authorization: `bearer ${this.Token.token}`,
          },
        })

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");

            // Si requête authentifiée
          } else {
            if (res.data.length > 0) {
              self.comments = res.data;
              self.showComments == true;
            } else {
              self.showComments == false;
            }
          }
        })
        .catch(function (err) {
          console.log("Erreur: ", err);
        });
    },
    // ------- Fin afficher tous les commentaires ------  //

    // -- 8/ -- Liker un post ---- //
    like_it(postId) {
      const self = this;
      axios
        .post(
          "http://localhost:3000/api/like/" + postId,
          {
            postId: postId,
            userId: this.userConnected.userId,
          },
          {
            headers: {
              authorization: `bearer ${this.Token.token}`,
            },
          }
        )

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.status = res.data.status;
            self.getAllMyPosts();
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    // --- Fin Liker un post ------- //

    // --9/ ----- Desliker un post ---- //
    deslike_it(postId) {
      const self = this;
      axios
        .post(
          "http://localhost:3000/api/deslike/" + postId,
          {
            postId: postId,
            userId: this.userConnected.userId,
          },
          {
            headers: {
              authorization: `bearer ${this.Token.token}`,
            },
          }
        )

        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.getAllMyPosts();
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    // --- Fin Desliker un post ------- //

    // -10/------Supprimer un commentaire
    deleteComment(commentId, postId) {
      const self = this;
      axios
        .delete(
          "http://localhost:3000/api/comments/" + `${commentId} ${postId}`,
          {
            headers: {
              authorization: `bearer ${this.Token.token}`,
            },
          }
        )
        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.okComment = false;
            self.getAllComments(postId);
            self.getAllMyPosts(postId);
          }
        })
        .catch(function (err) {
          console.log(err);
        });
    },
    // Fin Supprimer un commentaire

    // --11/ --- Masquer un commentaire par l'administrateur  -------- //
    maskComment(commentId, postId) {
      let self = this;
      this.okMasked = !this.okMasked;

      axios
        .patch("http://localhost:3000/api/comments/" + commentId, {
          headers: {
            authorization: `bearer ${this.Token.token}`,
          },
        })
        .then(function (res) {
          // Si requête non authentifiée
          if (res.data.disconnected) {
            this.authentified = res.data.disconnected;
            localStorage.removeItem("Token");
            this.$store.commit("DECONNEXION");
            // Si requête authentifiée
          } else {
            self.getAllComments(postId);
          }
        })
        .catch(function (err) {
          console.log("Erreur pour masquer un commentaire:", err);
        });
      this.getAllComments(postId);
      this.okMasked = false;
    },
    // ------ Fin masquer un commentaire par l'administrateur  ----//

    // --12/----- Montrer le commentaire masqué pour l'administrateur
    demaskComment(commentId) {
      console.log("commentId", commentId);
      this.okMasked = true;
    },

    // -----------fin methods  -----------//
  },
  ///// ------- Fin exports ----------
};
</script>

<style>
.deleteModify .delete,
.deleteModify .modify {
  padding-right: 10px;
}
</style>
