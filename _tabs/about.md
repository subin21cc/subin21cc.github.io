---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

<div class="about">
  <!-- ===== Intro ===== -->
  <section class="about-intro">
    <img class="about-avatar" src="{{ '/assets/img/profile_no_background.png' | relative_url }}" alt="Shin-SuBin" />
    <div class="about-intro-text">
      <h2 class="about-name">Shin-SuBin <span>신수빈</span></h2>
      <p class="about-affil">Ewha Womans Univ. · Computer Science</p>
      <p class="about-lead">배우고 만든 것을 코드와 함께 기록합니다.</p>
    </div>
  </section>

  <!-- ===== What I Work On ===== -->
  <section class="about-section">
    <h2 class="about-title">What I Work On</h2>
    <div class="about-stack">
      <div class="stack-row">
        <span class="stack-label">📱 Mobile · FE</span>
        <span class="stack-tags">
          <span class="tag">Flutter</span><span class="tag">Dart</span><span class="tag">React</span><span class="tag">TypeScript</span>
        </span>
      </div>
      <div class="stack-row">
        <span class="stack-label">🗄️ BE · Data</span>
        <span class="stack-tags">
          <span class="tag">Spring Boot</span><span class="tag">FastAPI</span><span class="tag">drift</span><span class="tag">SQLite</span><span class="tag">MySQL</span>
        </span>
      </div>
      <div class="stack-row">
        <span class="stack-label">🤖 AI · ML</span>
        <span class="stack-tags">
          <span class="tag">YOLOv8</span><span class="tag">Gemini Vision</span><span class="tag">LangChain</span><span class="tag">Pinecone</span><span class="tag">GPT-4o</span>
        </span>
      </div>
      <div class="stack-row">
        <span class="stack-label">⚙️ Workflow</span>
        <span class="stack-tags">
          <span class="tag">Git</span><span class="tag">GitHub Actions</span><span class="tag">Claude Code</span>
        </span>
      </div>
    </div>
  </section>

  <!-- ===== Blog Categories ===== -->
  <section class="about-section">
    <h2 class="about-title">Blog Categories</h2>
    <div class="about-cards">
      <a class="about-card" href="{{ '/categories/capstone/' | relative_url }}">
        <span class="about-card-head"><span class="about-card-icon">🎓</span> Capstone</span>
        <span class="about-card-desc">On-Care 졸업프로젝트 개발일지</span>
        <span class="about-card-meta">{{ site.categories['Capstone'] | size }} posts</span>
      </a>
      <a class="about-card" href="{{ '/categories/ecc/' | relative_url }}">
        <span class="about-card-head"><span class="about-card-icon">📚</span> ECC</span>
        <span class="about-card-desc">Ewha Computer Club — 깃 · SQL · 스프링 학습</span>
        <span class="about-card-meta">{{ site.categories['ECC'] | size }} posts</span>
      </a>
      <a class="about-card" href="{{ '/categories/dev/' | relative_url }}">
        <span class="about-card-head"><span class="about-card-icon">🧩</span> Dev</span>
        <span class="about-card-desc">블로그 운영 · 환경 설정 메타 글</span>
        <span class="about-card-meta">{{ site.categories['Dev'] | size }} posts</span>
      </a>
    </div>
    <p class="about-note">현재는 세 카테고리이지만, 활동 영역이 넓어지는 대로 새 카테고리를 점진적으로 늘려갈 예정입니다.</p>
  </section>

  <!-- ===== Contact ===== -->
  <section class="about-section">
    <h2 class="about-title">Contact</h2>
    <div class="about-contact">
      <a href="mailto:subin21cc@gmail.com"><i class="fas fa-envelope"></i> subin21cc@gmail.com</a>
      <a href="https://github.com/subin21cc" target="_blank" rel="noopener noreferrer"><i class="fab fa-github"></i> github.com/subin21cc</a>
    </div>
  </section>
</div>

<style>
  /* ===== Accent (matches home, mode-aware) ===== */
  .about {
    --about-accent: #2563eb;
    --about-accent-weak: rgba(37, 99, 235, 0.07);
    --about-muted: #6b7280;
  }
  html[data-mode='dark'] .about {
    --about-accent: #60a5fa;
    --about-accent-weak: rgba(96, 165, 250, 0.12);
    --about-muted: #9ca3af;
  }
  @media (prefers-color-scheme: dark) {
    html:not([data-mode='light']) .about {
      --about-accent: #60a5fa;
      --about-accent-weak: rgba(96, 165, 250, 0.12);
      --about-muted: #9ca3af;
    }
  }

  /* ===== Intro ===== */
  .about-intro {
    display: flex;
    align-items: center;
    gap: 1.2rem;
    padding-bottom: 1.8rem;
    border-bottom: 1px solid var(--main-border-color);
    margin-bottom: 0.4rem;
  }
  .about-avatar {
    width: 92px;
    height: 92px;
    border-radius: 50%;
    object-fit: cover;
    flex-shrink: 0;
    border: 1px solid var(--main-border-color);
    background: var(--card-bg);
  }
  .about-name {
    font-size: 1.45rem;
    font-weight: 700;
    letter-spacing: -0.01em;
    color: var(--heading-color);
    margin: 0 0 0.2rem;
  }
  .about-name span {
    font-size: 0.95rem;
    font-weight: 500;
    color: var(--about-muted);
    margin-left: 0.35rem;
  }
  .about-affil {
    font-size: 0.86rem;
    font-weight: 600;
    color: var(--about-accent);
    margin: 0 0 0.5rem;
  }
  .about-lead {
    font-size: 0.9rem;
    color: var(--about-muted);
    margin: 0;
    line-height: 1.5;
  }

  /* ===== Section ===== */
  .about-section { margin-top: 2.4rem; }
  .about-title {
    display: flex;
    align-items: center;
    gap: 0.55rem;
    font-size: 1.05rem;
    font-weight: 700;
    letter-spacing: -0.01em;
    color: var(--heading-color);
    margin: 0 0 1.1rem;
  }
  .about-title::before {
    content: '';
    width: 3px;
    height: 1.05em;
    border-radius: 2px;
    background: var(--about-accent);
  }

  /* ===== Tech stack ===== */
  .about-stack { display: flex; flex-direction: column; gap: 0.45rem; }
  .stack-row { display: flex; flex-wrap: wrap; align-items: center; gap: 0.3rem 0.55rem; }
  .stack-label {
    flex: 0 0 6rem;
    font-size: 0.84rem;
    font-weight: 600;
    color: var(--text-color);
  }
  .stack-tags { display: flex; flex-wrap: wrap; gap: 0.4rem; }
  .tag {
    font-size: 0.76rem;
    font-weight: 500;
    color: var(--text-color);
    background: var(--about-accent-weak);
    border: 1px solid transparent;
    padding: 0.13rem 0.5rem;
    border-radius: 0.45rem;
    transition: border-color 0.15s ease, color 0.15s ease;
  }
  .tag:hover { color: var(--about-accent); border-color: var(--about-accent); }

  /* ===== Category cards ===== */
  .about-cards {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 0.8rem;
  }
  .about-card {
    display: flex;
    flex-direction: column;
    padding: 1.05rem 1.15rem;
    border: 1px solid var(--main-border-color);
    border-radius: 0.85rem;
    background: var(--card-bg);
    text-decoration: none;
    transition: transform 0.18s ease, border-color 0.18s ease, box-shadow 0.18s ease;
  }
  .about-card:hover {
    transform: translateY(-3px);
    border-color: var(--about-accent);
    box-shadow: 0 6px 18px var(--about-accent-weak);
  }
  .about-card-head {
    font-size: 1rem;
    font-weight: 700;
    color: var(--heading-color);
    margin-bottom: 0.35rem;
  }
  .about-card-icon { margin-right: 0.15rem; }
  .about-card-desc {
    font-size: 0.83rem;
    color: var(--about-muted);
    line-height: 1.5;
    flex-grow: 1;
  }
  .about-card-meta {
    margin-top: 0.7rem;
    font-size: 0.74rem;
    font-weight: 600;
    letter-spacing: 0.02em;
    color: var(--about-accent);
  }
  .about-note {
    margin: 1rem 0 0;
    font-size: 0.83rem;
    color: var(--about-muted);
    line-height: 1.55;
  }

  /* ===== Contact ===== */
  .about-contact { display: flex; flex-wrap: wrap; gap: 0.6rem; }
  .about-contact a {
    display: inline-flex;
    align-items: center;
    gap: 0.45rem;
    font-size: 0.86rem;
    font-weight: 500;
    color: var(--text-color);
    text-decoration: none;
    border: 1px solid var(--main-border-color);
    background: var(--card-bg);
    padding: 0.45rem 0.9rem;
    border-radius: 0.6rem;
    transition: border-color 0.15s ease, color 0.15s ease, transform 0.15s ease;
  }
  .about-contact a:hover {
    border-color: var(--about-accent);
    color: var(--about-accent);
    transform: translateY(-1px);
  }
  .about-contact i { color: var(--about-accent); }

  /* ===== Mobile ===== */
  @media (max-width: 576px) {
    .about-intro { flex-direction: column; text-align: center; gap: 0.9rem; }
    .about-cards { grid-template-columns: 1fr; }
    .stack-label { flex-basis: auto; }
  }
</style>
