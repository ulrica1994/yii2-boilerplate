<template>
  <el-row :gutter="0" style="padding: 5px;">
    <div class="box-header">
      <h4>{{ pageTitle }}</h4>
    </div>
    <shop-category-form
      v-if="shopCategory"
      :submit-text="pageTitle"
      :shop-category="shopCategory"
      :category-tree="categoryTree"
      :category-parents="categoryParents"
      :is-edit="false"
      :can-submit="$can('shop-category/create')"
      @on-submit="createShopCategory"
    >
    </shop-category-form>
    <placeholder-form v-else></placeholder-form>
  </el-row>
</template>

<script>
import ShopCategoryForm from '@admin/views/shop-category/_EditForm'
import ShopCategoryService from '@admin/services/ShopCategoryService'
import MiscService from '@admin/services/MiscService'
import flatry from '@core/utils/flatry'
import PlaceholderForm from '@core/components/Placeholder/PlaceholderForm'

export default {
  name: 'ShopCategoryCreate',
  components: { PlaceholderForm, ShopCategoryForm },
  props: {
    categoryTree: {
      type: Array,
    },
  },
  data() {
    return {
      pageTitle: '新建商品分类',
      shopCategory: null,
      categoryParents: [],
    }
  },
  created() {
    this.getShopCategoryRoute(this.$route.params.id)
  },
  methods: {
    async getShopCategoryRoute(id) {
      let parents = [0]

      if (id > 0) {
        const { data } = await flatry(MiscService.shopCategoryPath(id))

        if (data && Array.isArray(data)) {
          parents = data.map((parent) => parent.id)
        }
      }

      this.categoryParents = parents

      this.$emit('on-expanded', parents, id)

      this.$nextTick(() => {
        this.shopCategory = {
          parentId: id,
          name: '',
          icon: '',
          image: '',
          description: '',
        }
      })
    },
    async createShopCategory(shopCategory, done, fail, always) {
      const { data, error } = await flatry(
        ShopCategoryService.create(shopCategory)
      )

      if (data) {
        done()

        this.$message.success('新建商品分类成功')

        this.$emit('on-updated', shopCategory.id)
      }

      if (error) {
        fail(error)
      }

      always()
    },
  },
}
</script>
