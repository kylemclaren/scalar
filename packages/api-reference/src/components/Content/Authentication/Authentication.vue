<script setup lang="ts">
import type { OpenAPIV3_1 } from 'openapi-types'
import { computed, watch } from 'vue'

import { hasSecuritySchemes } from '../../../helpers'
import { useGlobalStore } from '../../../stores'
import { type Spec } from '../../../types'
import { Card, CardContent, CardHeader } from '../../Card'
import SecurityScheme from './SecurityScheme.vue'
import SecuritySchemeSelector from './SecuritySchemeSelector.vue'

const props = defineProps<{ parsedSpec?: Spec }>()

const { authentication, setAuthentication } = useGlobalStore()

const showSecurityScheme = computed(() => {
  if (!authentication.securitySchemeKey) {
    return false
  }

  const scheme =
    props.parsedSpec?.components?.securitySchemes?.[
      authentication.securitySchemeKey
    ]

  return !!scheme && 'type' in scheme && !!scheme.type
})

// Keep a copy of the security schemes in the global authentication state
watch(
  () => props.parsedSpec?.components?.securitySchemes,
  () => {
    setAuthentication({
      securitySchemes: props.parsedSpec?.components?.securitySchemes,
    })
  },
  { deep: true, immediate: true },
)
</script>

<template>
  <Card v-if="hasSecuritySchemes(parsedSpec)">
    <CardHeader
      borderless
      class="authentication-header"
      transparent>
      Authentication
      <template #actions>
        <div class="selector">
          <SecuritySchemeSelector
            :value="
              parsedSpec?.components?.securitySchemes
            "></SecuritySchemeSelector>
        </div>
      </template>
    </CardHeader>
    <CardContent
      v-if="showSecurityScheme"
      class="authentication-content"
      transparent>
      <SecurityScheme
        v-if="authentication.securitySchemeKey"
        :value="
          parsedSpec?.components?.securitySchemes?.[
            authentication.securitySchemeKey
          ] as OpenAPIV3_1.SecuritySchemeObject
        " />
    </CardContent>
  </Card>
</template>
<style scoped>
.authentication-content {
  padding: 9px;
}
.selector {
  margin-right: 12px;
}
</style>
