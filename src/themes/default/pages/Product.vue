<template>
  <div id="product">
    <section class="bg-lightgray py30 px20 product-top-section">
      <div class="container">
        <breadcrumbs :routes="breadcrumbs.routes" :active-route="breadcrumbs.name"/>
        <section class="row py35 m0 data-wrapper">
          <div class="col-xs-12 col-md-7 center-xs middle-xs image">
            <transition name="fade" appear>
              <img class="product-image" v-lazy="imgObj" ref="image">
            </transition>
          </div>
          <div class="col-md-5 col-xs-12 px15 data">
            <div class="uppercase c-gray-secondary">
              sku: {{ product.sku}}
            </div>
            <h1 class="mb20 mt0 c-black product-name">
              {{ product.name | htmlDecode }}
            </h1>
            <div class="mb30 price" v-if="product.type_id !== 'grouped'">
              <div
                class="h3 c-gray-secondary"
                v-if="product.special_price && product.priceInclTax && product.originalPriceInclTax"
              >
                <span class="price-special">
                  {{ product.priceInclTax | price }}
                </span>&nbsp;
                <span class="price-original">
                  {{ product.originalPriceInclTax | price }}
                </span>
              </div>
              <div
                class="h3 c-gray"
                v-if="!product.special_price && product.priceInclTax"
              >
                {{ product.priceInclTax | price }}
              </div>
            </div>
            <div
              class="c-emperor variants"
              v-if="product.type_id =='configurable' && !loading"
            >
              <div
                class="h4"
                v-for="(option, index) in product.configurable_options"
                :key="index"
              >
                <div class="variants-label">
                  {{ option.label }}
                  <span class="weight-700">
                    {{ configuration[option.label.toLowerCase()].label }}
                  </span>
                </div>
                <div class="row top-xs m0 pt15 pb40 variants-wrapper">
                  <div v-if="option.label == 'Color'">
                    <color-button
                      v-for="(c, i) in options.color"
                      :key="i"
                      :id="c.id"
                      :label="c.label"
                      context="product"
                      code="color"
                      class="mr10"
                      :class="{ active: c.id == configuration.color.id }"
                    />
                  </div>
                  <div class="sizes" v-if="option.label == 'Size'">
                    <size-button
                      v-for="(s, i) in options.size"
                      :key="i"
                      :id="s.id"
                      :label="s.label"
                      context="product"
                      code="size"
                      class="mr10 mb10"
                      :class="{ active: s.id == configuration.size.id }"
                      v-focus-clean
                    />
                  </div>
                  <router-link
                    to="/size-guide"
                    v-if="option.label == 'Size'"
                    class="p0 ml30 no-underline action size-guide"
                  >
                    <i class="pr5 material-icons">accessibility</i>
                    <span>
                      Size guide
                    </span>
                  </router-link>
                </div>
              </div>
            </div>
            <product-links
              v-if="product.type_id =='grouped' && !loading"
              :products="product.product_links"
            />
            <div class="row m0">
              <add-to-cart
                :product="product"
                class="col-xs-12 col-sm-4 col-md-6 h4 bg-black c-white py20 brdr-none"
              />
            </div>
            <div class="row pt45 add-to-buttons">
              <div class="col-xs-12 col-sm-5">
                <button
                  @click="addToFavorite"
                  class="p0 bg-transparent brdr-none action"
                  type="button"
                >
                  <i class="pr5 material-icons">{{ favorite.icon }}</i>
                  Add to favorite
                </button>
              </div>
              <div class="hidden-xs col-md-7">
                <button
                  @click="addToCompare"
                  class="p0 bg-transparent brdr-none action"
                  type="button"
                >
                  <i class="pr5 material-icons">compare</i>
                    <template v-if="!compare.isCompare">
                      Add to compare
                    </template>
                    <template v-else>
                      Remove from compare
                    </template>
                </button>
              </div>
            </div>
          </div>
        </section>
      </div>
    </section>
    <section class="container pt50 pb20 px20 c-black details">
      <h2 class="h3 m0 mb10 sans-serif">
        Product details
      </h2>
      <div class="h4 details-wrapper" ref="details">
        <div class="row between-md m0">
          <div class="col-md-5">
            <div
              class="lh30 c-gray-secondary"
              v-html="product.description"
            ></div>
          </div>
          <div class="col-md-6">
            <ul class="attributes p0 pt10 m0">
              <product-attribute
                :key="attr.attribute_code"
                v-for="attr in all_custom_attributes"
                :product="product"
                :attribute="attr"
                emptyPlaceholder="N/A"
              ></product-attribute>
            </ul>
          </div>
          <div
            class="details-overlay"
            @click="showDetails"
          ></div>
        </div>
      </div>
    </section>
    <related-products/>
  </div>
</template>

<script>
import { corePage } from 'lib/themes'

import RelatedProducts from '../components/core/blocks/Product/Related.vue'
import AddToCart from '../components/core/AddToCart.vue'
import ColorButton from '../components/core/ColorButton.vue'
import SizeButton from '../components/core/SizeButton.vue'
import Breadcrumbs from '../components/core/Breadcrumbs.vue'
import ProductAttribute from '../components/core/ProductAttribute.vue'
import ProductTile from '../components/core/ProductTile.vue'
import ProductLinks from '../components/core/ProductLinks.vue'

export default {
  asyncData ({ store, route }) {
    // this is for SSR purposes to prefetch data
  },
  methods: {
    showDetails (event) {
      const details = this.$refs.details
      details.style.maxHeight = `${details.children[0].offsetHeight}px`
      event.target.classList.add('hidden')
    }
  },
  components: {
    AddToCart,
    ColorButton,
    SizeButton,
    Breadcrumbs,
    ProductAttribute,
    ProductTile,
    RelatedProducts,
    ProductLinks
  },
  mixins: [corePage('Product')]
}
</script>

<style lang="scss" scoped>
  .data-wrapper {
    @media (max-width: 767px) {
      padding: 0;
    }
  }

  .data {
    @media (max-width: 767px) {
      border-bottom: 1px solid #F2F2F2;
    }
  }

  .image {
    @media (max-width: 1023px) {
      margin-bottom: 20px;
      padding: 20px 0 30px 0;
      background-color: #F2F2F2;
    }
  }

  .product-name {
    @media (max-width: 767px) {
      margin-top: 10px;
      font-size: 36px;
    }
  }

  .price {
    @media (max-width: 767px) {
      color: #4F4F4F;
    }
  }

  .variants-label {
    @media (max-width: 767px) {
      font-size: 14px;
    }
  }

  .variants-wrapper {
    @media (max-width: 767px) {
      padding-bottom: 30px;
    }

   .sizes {
      @media (max-width: 767px) {
        width: 60%;
      }
    }

    .size-guide {
      height: 40px;
      @media (max-width: 767px) {
        width: 40%;
        margin-left: 0;
      }
    }
  }

  .product-top-section {
    @media (max-width: 767px) {
      padding: 0;
      background-color: #FFF;
    }
  }

  .add-to-buttons {
    @media (max-width: 767px) {
      padding-top: 30px;
      margin-bottom: 40px;
    }
  }

  .details {
    @media (max-width: 767px) {
      padding: 50px 15px 15px;
    }
  }

  .details-wrapper {
    @media (max-width: 767px) {
      position: relative;
      max-height: 140px;
      overflow: hidden;
      transition: all 0.3s ease;
      font-size: 14px;
    }
  }

  .details-overlay {
    @media (max-width: 767px) {
      position: absolute;
      height: 75%;
      bottom: 0;
      left: 0;
      width: 100%;
      margin: 0;
      cursor: pointer;
      background: linear-gradient(rgba(255, 255, 255, 0), rgba(255, 255, 255, 1));
      &.hidden {
        display: none;
      }
    }
  }

  .link-header {
    font-weight: bold;
  }

  .price-original {
    text-decoration: line-through;
    font-size: smaller;
  }

  .price-special {
    color: #FF0000;
  }

  .action {
    display: inline-flex;
    align-items: center;
    font-weight: 700;
    font-size: 14px;
    text-transform: uppercase;
    color: #BDBDBD;
    cursor: pointer;
  }

  .action:hover {
    color: #828282;
  }

  .attributes {
    list-style-type: none;
  }

  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s;
  }

  .fade-enter,
  .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }

  .product-image {
    display: inline-flex;
    mix-blend-mode: multiply;
    max-width: 100%;
    width: 460px;
  }
</style>
