<template>
  <div id="app" class="app-container">
    <div class="player">
      <h1 class="title">Reprodutor de Músicas</h1>
      <audio ref="audioPlayer" controls @ended="handleTrackEnded" @timeupdate="updateProgress">
        <source :src="currentTrack.source" type="audio/mpeg">
      </audio>
      <div class="player-controls">
        <button @click="playPrevTrack" class="control-button">Anterior</button>
        <button @click="startPlayback" class="control-button">Iniciar</button>
        <button @click="pausePlayback" v-if="isPlaying" class="control-button">Pausar</button>
        <button @click="resumePlayback" v-if="!isPlaying" class="control-button">Retomar</button>
        <button @click="playNextTrack" class="control-button">Próxima</button>
        <input type="range" min="0" :max="audioDuration" step="1" v-model="currentTime" @input="seekToTime"
          class="progress-bar" />
        <span class="time-display">{{ formatTime(currentTime) }} / {{ formatTime(audioDuration) }}</span>
        <input type="range" min="0" max="1" step="0.01" v-model="volume" @input="adjustVolume" class="volume-bar" />
        <button @click="toggleRepeat" class="control-button">Repetir: {{ repeat ? 'Ativado' : 'Desativado' }}</button>
        <button @click="toggleShuffle" class="control-button">Aleatório: {{ shuffle ? 'Ativado' : 'Desativado'
          }}</button>
      </div>
      <div class="album-art">
        <img :src="currentTrack.albumArt" alt="Capa do Álbum" v-if="currentTrack.albumArt" />
      </div>
      <div class="lyrics">
        <h2 class="lyrics-title">Letra da Música</h2>
        <p v-if="currentTrack.lyrics">{{ currentTrack.lyrics }}</p>
        <p v-else>Letra não disponível.</p>
      </div>
    </div>
    <div class="search">
      <input v-model="searchQuery" @input="searchTracks" placeholder="Pesquisar faixas..." class="search-input" />
    </div>
    <div class="playlist">
      <h2 class="playlist-title">Lista de Reprodução</h2>
      <button @click="addTrack" class="control-button">Adicionar Faixa</button>
      <ul>
        <li v-for="(track, index) in filteredTracks" :key="index" @click="playTrack(index)" class="playlist-item">
          <div class="track-info">
            <div class="title">{{ track.title }}</div>
            <div class="artist">{{ track.artist }}</div>
          </div>
          <button @click.stop="removeTrack(index)" class="remove-button">Remover</button>
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        currentTrackIndex: 0,
        tracks: [
          { title: 'Come As You Are', artist: 'Nirvana', source: 'assets/audio/Nirvana_Come_As_You_Are.mp3', albumArt: 'caminho/para/capa1.jpg', lyrics: 'Letra da música 1...' },
          { title: 'Smells Like Teen Spirit', artist: 'Nirvana', source: 'assets/audio/Nirvana_Smells_Like_Teen_Spirit.mp3', albumArt: 'caminho/para/capa2.jpg', lyrics: 'Letra da música 2...' },
          { title: 'Pump It', artist: 'The Black Eyed Peas', source: 'assets/audio/The_Black_Eyed_Peas_Pump_It.mp3', albumArt: 'caminho/para/capa2.jpg', lyrics: 'Letra da música 2...' },
        ],
        isPlaying: false,
        volume: 0.5,
        repeat: false,
        shuffle: false,
        currentTime: 0,
        audioDuration: 0,
        searchQuery: '',
      };
    },
    computed: {
      currentTrack() {
        return this.tracks[this.currentTrackIndex];
      },
      filteredTracks() {
        const query = this.searchQuery.toLowerCase();
        return this.tracks.filter((track) =>
          track.title.toLowerCase().includes(query) || track.artist.toLowerCase().includes(query)
        );
      },
    },
    methods: {
      startPlayback() {
        const audio = this.$refs.audioPlayer;
        audio.load();
        audio.play();
        this.isPlaying = true;
      },
      pausePlayback() {
        const audio = this.$refs.audioPlayer;
        audio.pause();
        this.isPlaying = false;
      },
      resumePlayback() {
        const audio = this.$refs.audioPlayer;
        audio.play();
        this.isPlaying = true;
      },
      playNextTrack() {
        if (this.shuffle) {
          this.currentTrackIndex = this.getRandomTrackIndex();
        } else if (this.repeat) {
          this.startPlayback(); // Repetir a faixa atual
        } else if (this.currentTrackIndex < this.filteredTracks.length - 1) {
          this.currentTrackIndex++;
        } else {
          this.currentTrackIndex = 0;
        }
        this.startPlayback();
      },
      playPrevTrack() {
        if (this.shuffle) {
          this.currentTrackIndex = this.getRandomTrackIndex();
        } else if (this.repeat) {
          this.startPlayback(); // Repetir a faixa atual
        } else if (this.currentTrackIndex > 0) {
          this.currentTrackIndex--;
        } else {
          this.currentTrackIndex = this.filteredTracks.length - 1;
        }
        this.startPlayback();
      },
      playTrack(index) {
        this.currentTrackIndex = index;
        this.startPlayback();
      },
      adjustVolume() {
        const audio = this.$refs.audioPlayer;
        audio.volume = this.volume;
      },
      toggleRepeat() {
        this.repeat = !this.repeat;
      },
      toggleShuffle() {
        this.shuffle = !this.shuffle;
      },
      handleTrackEnded() {
        if (this.repeat) {
          this.startPlayback(); // Repetir a faixa atual
        } else {
          this.playNextTrack();
        }
      },
      updateProgress() {
        const audio = this.$refs.audioPlayer;
        this.currentTime = audio.currentTime;
        this.audioDuration = audio.duration;
      },
      seekToTime() {
        const audio = this.$refs.audioPlayer;
        audio.currentTime = this.currentTime;
      },
      formatTime(seconds) {
        const minutes = Math.floor(seconds / 60);
        const remainingSeconds = Math.floor(seconds % 60);
        return `${minutes}:${remainingSeconds < 10 ? '0' : ''}${remainingSeconds}`;
      },
      getRandomTrackIndex() {
        const currentIndex = this.currentTrackIndex;
        let randomIndex;
        do {
          randomIndex = Math.floor(Math.random() * this.filteredTracks.length);
        } while (randomIndex === currentIndex);
        return randomIndex;
      },
      addTrack() {
        this.tracks.push({
          title: 'Nova Música',
          artist: 'Novo Artista',
          source: 'caminho/para/nova_musica.mp3',
          albumArt: 'caminho/para/nova_capa.jpg',
        });
      },
      removeTrack(index) {
        this.tracks.splice(index, 1);
      },
      searchTracks() {
        // Realizar a pesquisa em tempo real
      },
    },
  };
</script>
<style>
  /* Estilos para o contêiner principal */
  .app-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
  }

  /* Estilos para o reprodutor de música */
  .player {
    background-color: #f2f2f2;
    padding: 20px;
    border: 1px solid #ccc;
    text-align: center;
  }

  /* Estilos para o título do reprodutor de música */
  .title {
    font-size: 24px;
    margin-bottom: 20px;
  }

  /* Estilos para os controles do reprodutor de música */
  .player-controls {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    margin: 20px 0;
  }

  .control-button {
    background-color: #007bff;
    color: #fff;
    border: none;
    padding: 10px 20px;
    margin: 5px;
    cursor: pointer;
    border-radius: 5px;
  }

  .control-button:hover {
    background-color: #0056b3;
  }

  .progress-bar {
    width: 100%;
  }

  .time-display {
    font-size: 14px;
    margin-top: 10px;
  }

  .volume-bar {
    width: 100%;
    margin-top: 10px;
  }

  /* Estilos para a capa do álbum */
  .album-art {
    margin-top: 20px;
  }

  .album-art img {
    max-width: 100%;
  }

  /* Estilos para as letras da música */
  .lyrics {
    margin-top: 20px;
  }

  .lyrics-title {
    font-size: 20px;
  }

  /* Estilos para a caixa de pesquisa */
  .search {
    margin-top: 20px;
  }

  .search-input {
    width: 100%;
    padding-top: 10px;
    padding-bottom: 10px;
    padding-right: -100px;
    font-size: 16px;
  }

  /* Estilos para a lista de reprodução */
  .playlist {
    margin-top: 20px;
  }

  .playlist-title {
    font-size: 20px;
    margin-bottom: 10px;
  }

  .playlist-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 0;
    border-bottom: 1px solid #ccc;
  }

  .track-info {
    flex: 1;
  }

  .title {
    font-weight: bold;
  }

  .artist {
    font-style: italic;
  }

  .remove-button {
    background-color: #ff5050;
    color: #fff;
    border: none;
    padding: 5px 10px;
    cursor: pointer;
    border-radius: 5px;
  }

  .remove-button:hover {
    background-color: #d9534f;
  }
</style>
