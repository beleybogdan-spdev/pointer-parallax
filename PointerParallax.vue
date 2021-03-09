<template>
  <div class="pointer_parallax">
    <slot />
  </div>
</template>

<script>
export default {
  props: {
    elementWrapper: {
      type: [Object, String],
      default: undefined
    },
    smooth: {
      type: Boolean,
      default: true,
    },
    smoothDuration: {
      type: Number,
      default: 100,
    },
    mode: {
      default: 'reverse'
    },
    coef: {
      type: Number,
      default: 0.00025
    },
  },
  data() {
    return {
      elms: [],
      el: false,
      smoothTimeout: false,
      smoothTimeoutEvent: false,
      levels: 1,
      levelsElements: {},
    };
  },
  mounted() {
    if (this.elementWrapper && this.elementWrapper.addEventListener) {
      this.el = this.elementWrapper;
    } else if (this.elementWrapper == 'self') {
      this.el = this.$el;
    } else if (typeof this.elementWrapper == 'string') {
      this.el = document.querySelector(this.elementWrapper);
    } else {
      this.el = document;
    }

    // Add event listeners
    if (this.smooth) {
      this.el.addEventListener("mouseleave", this.parallaxRemoveStyles);
      this.el.addEventListener("mousemove", this.parallaxSmooth);
    } else {
      this.el.addEventListener("mousemove", this.parallax);
    }

    // List paralax elements
    let elms = this.$el.querySelectorAll(".pointer_parallax__element");
    elms.forEach(el => {
      let level = el.dataset.level ? +el.dataset.level : 1;

      this.elms.push({
        level,
        el
      });
    });

    if (this.smooth) {
      this.addSmoothTransition();
    }
  },

  beforeDestroy() {
    // Remove event listeners
    if (this.smooth) {
      this.el.removeEventListener("mouseleave", this.parallaxRemoveStyles);
      this.el.removeEventListener("mousemove", this.parallaxSmooth);
    } else {
      this.el.removeEventListener("mousemove", this.parallax);
    }
  },

  methods: {
    parallaxRemoveStyles () {
      this.elms.forEach(e => {
        if (e.el) e.el.style.transform = null;
      });
    },

    addSmoothTransition () {
      this.elms.forEach(e => {
        e.el.style.transition = `transform ${this.smoothDuration}ms ease-out`;
      });
    },

    parallax(e) {
      let depth = (this.mode == 'reverse') ? this.getDepthReverse(e) : this.getDepth(e);

      this.elms.forEach(e => {
        if (e.el) e.el.style.transform = `translate(${depth[e.level]})`;
      });
    },

    parallaxSmooth(e) {
      if (!this.smoothTimeout) {
        let depth = (this.mode == 'reverse') ? this.getDepthReverse(e) : this.getDepth(e);
        this.elms.forEach(e => {
          if (e.el) e.el.style.transform = `translate(${depth[e.level]})`;
        });
        this.smoothTimeout = setTimeout(() => {
            this.smoothTimeout = null;
        }, this.smoothDurationDelay);
      } else {
        this.smoothTimeoutEvent = e;
      }
    },

    getHeight () {
      return this.el != document ? this.el.offsetHeight : window.innerHeight;
    },
    getWidth () {
      return this.el != document ? this.el.offsetWidth : window.innerWidth;
    },

    getMousePosition (e) {
      let _mouseX = e.clientX;
      let _mouseY = e.clientY;
      let _w,
          _h,
          _mx,
          _my;

      if (this.el) {
        let rect = this.el.getBoundingClientRect();
        _w = rect.width / 2;
        _h = rect.height / 2;
        _mx = (_mouseX - rect.x) - _w;
        _my = (_mouseY - rect.y) - _h;
      } else {
        _w = window.innerWidth / 2;
        _h = window.innerHeight / 2;
        _mx = _mouseX - _w;
        _my = _mouseY - _h;
      }
      return {
        x: _mx,
        y: _my
      }
    },

    getDepthObj (e) {
      let _m = this.getMousePosition(e);

      let depth = {};
      for (let i = 0; i < this.level; i++) {
        depth[i] = {
          x: (_m.x * (this.coef * i)).toFixed(5),
          y: (_m.y * (this.coef * i)).toFixed(5)
        };
      }
      return depth;
    },

    getDepth (e) {
      let depth = this.getDepthObj(e);

      for (let i = 0; i < this.level; i++) {
        depth[i] = `${depth[i].x}%, ${depth[i].y}%`;
      }

      return depth;
    },

    getDepthReverse (e) {
      let depth = this.getDepthObj(e);

      for (let i = 0; i < this.level; i++) {
        depth[i] = `${0 - depth[i].x}%, ${0 - depth[i].y}%`;
      }

      return depth;
    }
  },

  computed: {
    smoothDurationDelay () {
      return this.smoothDuration / 2;
    }
  },
};
</script>