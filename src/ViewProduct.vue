<template>
  <div>
    <button class="btn btn-primary" @click="goBack">&larr; Back</button>
    <div v-if="product!=null">
      <h1>{{product.name}}</h1>
      <p><strong>ID: </strong>{{product.id}}</p>
      <p><strong>Price: </strong>{{product.price | currency}}</p>
      <p><strong>In stock: </strong>{{product.inStock}}</p>
      <p>{{product.description}}</p>
      <h2>Reviews</h2>
      <div v-if="product.reviews.length > 0">
        <div v-for="review in product.reviews" :key="review.id">
          <strong>{{review.reviewer}}</strong> (rating: {{review.rating}})
          <button class="btn btn-danger btn-xs"
                  @click.prevent="deleteReview(review)">delete
          </button>
          <p>{{review.text}}</p>
        </div>
      </div>
      <div v-else>
        <p>No reviews have been added for this product.</p>
      </div>
      <h3>Add Review</h3>
      <form @submit.prevent="addNewReview(newReview)">
        <div class="form-group">
          <label for="reviewName">Name</label>
          <input id="reviewName" type="text" class="form-control"
                 placeholder="Name"
                 v-model="newReview.reviewer">
        </div>
        <div class="form-group">
          <label for="reviewRating">Rating</label>
          <input id="reviewRating" type="number" class="form-control"
                 placeholder="Rating"
                 v-model.number="newReview.rating">
        </div>
        <div class="form-group">
          <label for="reviewText">Text</label>
          <textarea id="reviewText" class="form-control" rows="7"
                    v-model="newReview.text">
          </textarea>
        </div>
        <button type="submit" class="btn btn-primary">Submit Review</button>
      </form>
    </div>
    <div v-else>
      <p>Loading...</p>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      productId: {required: true},
    },
    data() {
      return {
        product: null,
        newReview: {
          text: '',
          rating: 0,
          reviewer: '',
        },
        reviewResource: null,
      };
    },
    created() {
      let main_url = 'products/{productId}/reviews/{reviewId}';
      let customActions = {
        softDelete: {
          method: 'Delete',
          url: main_url + '?soft=true',
        },
      };
      this.reviewResource = this.$resource(main_url, {}, customActions);
      this.getProduct(this.productId)
          .then(product => this.product = product);
    },
    beforeRouteUpdate(to, from, next) {
      this.getProduct(to.params.productId)
          .then(product => this.product = product);
      next();
    },
    methods: {
      getProduct(productId) {
        return this.$http.get('products/{productId}', {params: {productId}})
                         .then(response => response.json(),
                               response => alert('Could not retrieve product.'));
      },
      goBack() {
        this.$router.history.go(-1);
      },
      addNewReview(review) {
        this.$http.post('products/{productId}/reviews', review, {
                    params: {productId: this.product.id}})
                  .then(response => response.json(),
                        response => alert("Could not add review!"))
                  .then(newReview => this.product.reviews.push(newReview));
      },
      deleteReview(rev) {
        // this.reviewResource.delete({productId: this.product.id,
        this.reviewResource.softDelete({productId: this.product.id,
                                        reviewId: rev.id})
                           .then(response => {
                                   this.getProduct(this.product.id)
                                       .then(product => this.product = product)},
                                 response => alert("Could not delete review!"));
      },
    },
  };
</script>

<style scoped>
  textarea {resize: none;}
</style>
