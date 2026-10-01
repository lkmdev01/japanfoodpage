<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref } from "vue";
import {
  ArrowUpRight,
  ArrowRight,
  ArrowDown,
  Search,
  ShoppingBag,
  Plus,
  Minus,
  X,
  Menu,
  Send,
  Check,
  Leaf,
} from "lucide-vue-next";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);
const root = ref<HTMLElement>();
const cartDialog = ref<HTMLDialogElement>();
const menuOpen = ref(false);
const search = ref("");
const email = ref("");
const emailMessage = ref("");
const toast = ref("");
let toastTimeout: ReturnType<typeof setTimeout> | undefined;
const dishes = [
  {
    id: 1,
    name: "Trio de Nigiris",
    description: "Salmão, atum e camarão. Três encontros com o sabor do Japão.",
    pieces: "3 unidades",
    price: 32.9,
    image: "/images/hero.webp",
    alt: "Nigiris de salmão, atum e camarão em uma bandeja preta",
    tag: "O favorito",
  },
  {
    id: 2,
    name: "Seleção Kami",
    description:
      "Makis delicados, peixes frescos e o equilíbrio perfeito em cada peça.",
    pieces: "12 unidades",
    price: 54.9,
    image: "/images/maki.webp",
    alt: "Makis de salmão, ovas e atum com avocado",
    tag: "",
  },
  {
    id: 3,
    name: "Maki Salmon",
    description: "Salmão e avocado, envoltos em arroz japonês e alga nori.",
    pieces: "8 unidades",
    price: 38.9,
    image: "/images/feature.webp",
    alt: "Maki de salmão e avocado servido com shoyu",
    tag: "",
  },
];
const quantities = ref<Record<number, number>>({});
const normalize = (value: string) =>
  value
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .toLowerCase();
const visibleDishes = computed(() =>
  dishes.filter((dish) =>
    normalize(dish.name + dish.description).includes(normalize(search.value)),
  ),
);
const cartItems = computed(() =>
  dishes.filter((dish) => (quantities.value[dish.id] ?? 0) > 0),
);
const cartCount = computed(() =>
  Object.values(quantities.value).reduce((sum, qty) => sum + qty, 0),
);
const total = computed(() =>
  cartItems.value.reduce(
    (sum, dish) => sum + dish.price * (quantities.value[dish.id] ?? 0),
    0,
  ),
);
const money = (value: number) =>
  new Intl.NumberFormat("pt-BR", { style: "currency", currency: "BRL" }).format(
    value,
  );
function addToCart(id: number) {
  quantities.value[id] = (quantities.value[id] ?? 0) + 1;
  toast.value = "Adicionado à sua sacola";
  clearTimeout(toastTimeout);
  toastTimeout = setTimeout(() => {
    toast.value = "";
  }, 2400);
}
function changeQuantity(id: number, delta: number) {
  quantities.value[id] = Math.max(0, (quantities.value[id] ?? 0) + delta);
}
function openCart() {
  cartDialog.value?.showModal();
  document.body.style.overflow = "hidden";
}
function closeCart() {
  cartDialog.value?.close();
  document.body.style.overflow = "";
}
function goToMenu() {
  document
    .querySelector("#cardapio")
    ?.scrollIntoView({
      behavior: window.matchMedia("(prefers-reduced-motion: reduce)").matches
        ? "instant"
        : "smooth",
    });
  menuOpen.value = false;
}
function subscribe() {
  emailMessage.value =
    "As inscrições abrem em breve. Volte para descobrir nossas novidades!";
}
let media: gsap.MatchMedia | undefined;
onMounted(async () => {
  await nextTick();
  media = gsap.matchMedia();
  media.add(
    "(prefers-reduced-motion: no-preference)",
    () => {
      gsap.from(".hero-copy > *", {
        y: 25,
        opacity: 0,
        duration: 1,
        stagger: 0.13,
        ease: "power3.out",
        clearProps: "all",
      });
      gsap.from(".hero-food", {
        x: 65,
        y: -15,
        rotation: -4,
        opacity: 0,
        duration: 1.4,
        ease: "power3.out",
        clearProps: "all",
      });
      gsap.utils.toArray<HTMLElement>("[data-reveal]").forEach((element) => {
        gsap.from(element, {
          y: 35,
          opacity: 0,
          duration: 0.9,
          ease: "power2.out",
          scrollTrigger: { trigger: element, start: "top 92%", once: true },
          clearProps: "all",
        });
      });
      gsap.utils
        .toArray<HTMLElement>("[data-float]")
        .forEach((element, index) => {
          gsap.to(element, {
            y: index % 2 ? 10 : -10,
            rotation: index % 2 ? 5 : -5,
            duration: 3 + index * 0.2,
            repeat: -1,
            yoyo: true,
            ease: "sine.inOut",
          });
        });
      gsap.to(".seal-ring", {
        rotation: 360,
        duration: 40,
        repeat: -1,
        ease: "none",
        transformOrigin: "50% 50%",
      });
    },
    root.value,
  );
});
onUnmounted(() => {
  media?.revert();
  clearTimeout(toastTimeout);
  document.body.style.overflow = "";
});
</script>

<template>
  <div ref="root" class="site-shell">
    <a class="skip-link" href="#conteudo">Pular para o conteúdo</a>
    <header class="header page-width">
      <nav class="desktop-nav" aria-label="Navegação principal">
        <a href="#cardapio">Cardápio</a><a href="#sobre">Nossa essência</a
        ><a href="#especial">Especial da casa</a>
      </nav>
      <a class="wordmark" href="#" aria-label="Kami Sushi, início"
        >kami<span class="brand-dot">.</span
        ><span class="wordmark-caption">JAPANESE SOUL</span></a
      >
      <div class="header-actions">
        <form class="search-form" role="search" @submit.prevent="goToMenu">
          <input
            v-model="search"
            type="search"
            aria-label="Buscar no cardápio"
            placeholder="Encontre seu favorito"
          />
          <button type="submit" aria-label="Buscar">
            <Search :size="16" />
          </button>
        </form>
        <button class="cart-button" aria-label="Abrir sacola" @click="openCart">
          <span class="bag-icon"
            ><ShoppingBag :size="21" :stroke-width="1.5" /><span
              class="cart-count"
              >{{ cartCount }}</span
            ></span
          ><span class="cart-label"
            >Sua sacola<small>{{ money(total) }}</small></span
          >
        </button>
        <button
          class="mobile-toggle icon-button"
          :aria-expanded="menuOpen"
          aria-controls="mobile-menu"
          aria-label="Abrir menu"
          @click="menuOpen = !menuOpen"
        >
          <X v-if="menuOpen" :size="23" /><Menu v-else :size="23" />
        </button>
      </div>
      <nav
        v-if="menuOpen"
        id="mobile-menu"
        class="mobile-nav"
        aria-label="Navegação móvel"
      >
        <a href="#cardapio" @click="menuOpen = false">Cardápio</a
        ><a href="#sobre" @click="menuOpen = false">Nossa essência</a
        ><a href="#especial" @click="menuOpen = false">Especial da casa</a>
      </nav>
    </header>

    <main id="conteudo">
      <section class="hero page-width" aria-labelledby="hero-heading">
        <div class="hero-copy">
          <p class="eyebrow">
            <span></span> FEITO COM ALMA. SERVIDO COM AFETO.
          </p>
          <h1 id="hero-heading">O Japão em<br />cada <em>detalhe.</em></h1>
          <p class="body-copy">
            Pequenos encontros. Grandes sabores.<br />Descubra o prazer de um
            sushi feito para sentir.
          </p>
          <a class="pill-button" href="#cardapio"
            >Explore o cardápio <ArrowUpRight :size="19"
          /></a>
          <div class="hero-note">
            <span class="little-line"></span> Frescor que se sente. Cuidado que
            se prova.
          </div>
        </div>
        <div class="hero-visual">
          <span class="kanji" aria-hidden="true">旨い</span>
          <img
            class="hero-food"
            src="/images/hero.webp"
            width="1536"
            height="1024"
            alt="Três nigiris de salmão, atum e camarão, servidos em uma bandeja com hashis"
            fetchpriority="high"
          />
          <span class="visual-caption"
            >伝統と情熱 <span>TRADIÇÃO & PAIXÃO</span></span
          >
        </div>
        <a class="scroll-seal" href="#sobre" aria-label="Conheça nossa essência"
          ><svg class="seal-ring" viewBox="0 0 120 120" aria-hidden="true">
            <defs>
              <path
                id="sealPath"
                d="M60 60m-46 0a46 46 0 1 1 92 0a46 46 0 1 1-92 0"
              />
            </defs>
            <text>
              <textPath href="#sealPath" textLength="284">
                SABOREIE O MOMENTO · FEITO COM ALMA ·
              </textPath>
            </text></svg
          ><span><ArrowDown :size="20" :stroke-width="1.5" /></span
        ></a>
        <span
          class="floating-leaf hero-leaf"
          data-float
          aria-hidden="true"
        ></span>
      </section>

      <section
        id="sobre"
        class="about page-width section-grid"
        aria-labelledby="about-heading"
      >
        <div class="about-visual" data-reveal>
          <svg class="contour" viewBox="0 0 210 210" aria-hidden="true">
            <g fill="none" stroke="currentColor" stroke-width="1.2">
              <path
                v-for="n in 8"
                :key="n"
                :d="`M ${15 + n * 8} 180 C ${-30 + n * 10} ${80 + n * 4}, ${170 - n * 8} ${170 - n * 11}, ${70 + n * 10} ${10 + n * 5} S 240 100, 190 170`"
              />
            </g>
          </svg>
          <img
            src="/images/maki.webp"
            width="1536"
            height="1024"
            loading="lazy"
            alt="Trio de makis preparados com salmão, ovas e avocado"
          />
          <span class="red-pearl pearl-one" data-float aria-hidden="true"></span
          ><span
            class="red-pearl pearl-two"
            data-float
            aria-hidden="true"
          ></span>
          <span class="image-caption"
            >Simples na essência. Extraordinário no sabor.</span
          >
        </div>
        <div class="section-copy" data-reveal>
          <span class="script-label">Nossa essência</span>
          <h2 id="about-heading">
            Bom para o corpo.<br /><em>Perfeito para a alma.</em>
          </h2>
          <p class="body-copy">
            Acreditamos que a boa comida vai além do paladar. Ela conta
            histórias, aproxima pessoas e transforma o dia.
          </p>
          <p class="body-copy">
            Por isso, cada peça leva ingredientes frescos, respeito à tradição
            japonesa e um toque do nosso coração.
          </p>
          <a class="text-button" href="#cardapio"
            ><span class="round-arrow"><ArrowRight :size="16" /></span> Descubra
            nossos sabores</a
          >
        </div>
        <svg
          class="botanical botanical-about"
          viewBox="0 0 120 220"
          fill="none"
          aria-hidden="true"
        >
          <path
            d="M85 210C70 155 75 90 32 10M72 158C8 149 1 112 19 121C40 132 46 124 65 137M65 119C101 71 122 75 107 98C94 121 93 131 71 139M51 74C4 80 3 45 17 53C33 66 36 48 46 61M40 42C62 5 78 4 69 29C58 43 59 51 49 62"
            stroke="currentColor"
            stroke-width="1.5"
          />
        </svg>
      </section>

      <section
        id="cardapio"
        class="menu-section page-width"
        aria-labelledby="menu-heading"
      >
        <div class="section-heading" data-reveal>
          <span class="eyebrow">ESCOLHIDOS PARA VOCÊ</span>
          <h2 id="menu-heading">Pequenas obras. <em>Grandes sabores.</em></h2>
          <p>Nossos favoritos, preparados na hora para o seu momento.</p>
        </div>
        <div v-if="search" class="search-summary" aria-live="polite">
          {{ visibleDishes.length }} resultado(s) para “{{ search }}”
          <button @click="search = ''">Limpar busca <X :size="14" /></button>
        </div>
        <div class="dish-grid">
          <article
            v-for="dish in visibleDishes"
            :key="dish.id"
            class="dish-card"
            data-reveal
          >
            <span v-if="dish.tag" class="dish-tag">{{ dish.tag }}</span>
            <div class="dish-image">
              <img
                :src="dish.image"
                :alt="dish.alt"
                width="1536"
                height="1024"
                loading="lazy"
              />
            </div>
            <div class="dish-info">
              <span class="dish-pieces">{{ dish.pieces }}</span>
              <h3>{{ dish.name }}</h3>
              <p>{{ dish.description }}</p>
              <div class="dish-bottom">
                <span class="price">{{ money(dish.price) }}</span
                ><button
                  class="add-button"
                  :aria-label="`Adicionar ${dish.name} à sacola`"
                  @click="addToCart(dish.id)"
                >
                  <ShoppingBag :size="17" :stroke-width="1.5" /><Plus
                    :size="10"
                    class="bag-plus"
                  />
                </button>
              </div>
            </div>
          </article>
        </div>
        <p v-if="!visibleDishes.length" class="empty-search">
          Ainda não encontramos esse sabor. Experimente buscar por salmão, maki
          ou nigiri.
        </p>
        <div class="menu-footnote">
          <span></span><Leaf :size="14" /> Ingredientes frescos. Sempre.<span
          ></span>
        </div>
      </section>

      <section
        id="especial"
        class="special page-width section-grid"
        aria-labelledby="special-heading"
      >
        <div class="section-copy" data-reveal>
          <span class="script-label">Um novo favorito</span>
          <h2 id="special-heading">
            Maki Salmon.<br /><em>Amor à primeira peça.</em>
          </h2>
          <p class="body-copy">
            A delicadeza do salmão, a cremosidade do avocado e aquele mergulho
            no shoyu. Tem combinações que a gente nunca esquece.
          </p>
          <button class="pill-button" @click="addToCart(3)">
            Quero experimentar <ArrowUpRight :size="19" /></button
          ><span class="special-detail"
            >8 peças de felicidade <span>·</span> {{ money(38.9) }}</span
          >
        </div>
        <div class="special-visual" data-reveal>
          <img
            src="/images/feature.webp"
            alt="Maki de salmão e avocado entre hashis sobre uma tigela de shoyu"
            width="1536"
            height="1024"
            loading="lazy"
          />
        </div>
        <span
          class="floating-leaf special-leaf"
          data-float
          aria-hidden="true"
        ></span>
      </section>

      <section
        id="novidades"
        class="newsletter page-width"
        aria-labelledby="newsletter-heading"
        data-reveal
      >
        <div class="newsletter-art">
          <img
            src="/images/hero.webp"
            alt=""
            width="1536"
            height="1024"
            loading="lazy"
          />
        </div>
        <div class="newsletter-copy">
          <span class="script-label">Fique por perto</span>
          <h2 id="newsletter-heading">
            O próximo bom momento<br />pode chegar por e-mail.
          </h2>
          <form class="newsletter-form" @submit.prevent="subscribe">
            <label class="sr-only" for="email">Seu melhor e-mail</label
            ><input
              id="email"
              v-model="email"
              type="email"
              placeholder="Seu melhor e-mail"
              required
              maxlength="254"
              autocomplete="email"
            /><button type="submit">Me avise <Send :size="15" /></button>
          </form>
          <p v-if="emailMessage" class="form-message" role="status">
            {{ emailMessage }}
          </p>
          <p v-else class="newsletter-note">
            Novos sabores e novidades. Só o que vale saborear.
          </p>
        </div>
        <span class="newsletter-spark" aria-hidden="true">✳</span>
      </section>
    </main>

    <footer class="footer page-width">
      <div class="footer-main">
        <div class="footer-brand">
          <a class="wordmark" href="#" aria-label="Kami Sushi, início"
            >kami<span class="brand-dot">.</span
            ><span class="wordmark-caption">JAPANESE SOUL</span></a
          >
          <p>Para alimentar o corpo.<br />E deixar a alma feliz.</p>
        </div>
        <div class="footer-column">
          <h3>Explore</h3>
          <a href="#sobre">Nossa essência</a><a href="#cardapio">Cardápio</a
          ><a href="#especial">Especial da casa</a>
        </div>
        <div class="footer-column">
          <h3>Vamos nos aproximar</h3>
          <a href="#novidades">Receba novidades</a
          ><button @click="openCart">
            Sua sacola <ArrowUpRight :size="12" /></button
          ><a href="#cardapio">Encontre seu favorito</a>
        </div>
        <div class="footer-manifesto">
          <span class="footer-kanji" aria-hidden="true">心を込めて</span>
          <p>Feito com cuidado.<br />Saboreado sem pressa.</p>
        </div>
      </div>
      <div class="footer-bottom">
        <span
          >© {{ new Date().getFullYear() }} Kami Sushi. Feito com alma.</span
        ><span class="footer-bottom-note"
          >Tradição japonesa. Um toque de Kami.<span class="tiny-dot"></span
        ></span>
      </div>
    </footer>
    <Transition name="toast"
      ><div v-if="toast" class="toast-message" role="status">
        <Check :size="17" />{{ toast
        }}<button @click="openCart">
          Ver sacola <ArrowRight :size="14" />
        </button></div
    ></Transition>
    <dialog
      ref="cartDialog"
      class="cart-dialog"
      aria-labelledby="cart-heading"
      @click="
        (event) => {
          if (event.target === cartDialog) closeCart();
        }
      "
      @close="closeCart"
      @cancel="closeCart"
    >
      <div class="cart-inner">
        <div class="cart-heading">
          <div>
            <span class="eyebrow">SEU MOMENTO KAMI</span>
            <h2 id="cart-heading">
              Sua sacola<span class="brand-dot">.</span>
            </h2>
          </div>
          <button
            class="icon-button"
            aria-label="Fechar sacola"
            @click="closeCart"
          >
            <X />
          </button>
        </div>
        <div v-if="!cartItems.length" class="cart-empty">
          <ShoppingBag :size="46" :stroke-width="1" />
          <h3>Um bom momento começa aqui.</h3>
          <p>Escolha seus favoritos no cardápio.</p>
          <button
            class="pill-button"
            @click="
              closeCart();
              goToMenu();
            "
          >
            Explorar sabores <ArrowRight :size="17" />
          </button>
        </div>
        <template v-else
          ><div class="cart-lines">
            <article v-for="dish in cartItems" :key="dish.id" class="cart-line">
              <img :src="dish.image" :alt="dish.name" />
              <div>
                <h3>{{ dish.name }}</h3>
                <span>{{ money(dish.price) }}</span>
                <div class="quantity-control">
                  <button
                    :aria-label="`Remover uma unidade de ${dish.name}`"
                    @click="changeQuantity(dish.id, -1)"
                  >
                    <Minus :size="14" /></button
                  ><span :aria-label="`${quantities[dish.id]} unidades`">{{
                    quantities[dish.id]
                  }}</span
                  ><button
                    :aria-label="`Adicionar uma unidade de ${dish.name}`"
                    @click="changeQuantity(dish.id, 1)"
                  >
                    <Plus :size="14" />
                  </button>
                </div>
              </div>
              <strong>{{
                money(dish.price * (quantities[dish.id] ?? 0))
              }}</strong>
            </article>
          </div>
          <div class="cart-summary">
            <span>Subtotal</span><strong>{{ money(total) }}</strong>
          </div>
          <p class="cart-notice">
            Os pedidos on-line estarão disponíveis em breve. Por enquanto,
            explore e monte sua seleção favorita.
          </p>
          <button
            class="pill-button"
            @click="
              closeCart();
              goToMenu();
            "
          >
            Continuar explorando <ArrowRight :size="17" /></button
        ></template>
      </div>
    </dialog>
  </div>
</template>
